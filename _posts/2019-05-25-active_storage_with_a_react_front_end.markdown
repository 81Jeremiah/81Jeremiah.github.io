---
layout: post
title:      "Active Storage with a React Front End"
date:       2019-05-25 13:01:40 +0000
permalink:  active_storage_with_a_react_front_end
---


For my final project at Flatiron school I made an online database for web influencers/content creators. In addition to requirements listed in the curriculum it was very important to have the ability for a user to upload photos. In my head this sounded like an easy feature to add, ultimately it was but I ran into a few issues that I would like to share in the hope it might help others as they venture into this task. 

My first question was how do I store images in the database?  When I started, I thought that somehow Rails would magically turn the image into a string and store in SQLite with the rest of the data. I was very wrong about this, but it turns out Rails 5.2 and above does come with a very handy feature ACTIVE STORAGE. 

My second question was how do I set the image in state and then send it with the rest of my data to Active Storage? After trying to just set the html form type to “file” , setting the state to the value of that file and crashing the app, I decided that there must be a better way to do this.  After some searching and a few more blog posts I decided to use React Dropzone for this app. Dropzone is a “Simple React hook to create a HTML5-compliant drag'n'drop zone for files”. Basically React Dropzone is a node package that easily takes care a lot of the issues that arise when uploading a file with React. 

**Installing Active Storage**

If you are one of those people who always updates, chances are you're already using Rails 5.2 and can skip ahead.  But if you are like me and accidently created the app in rails 5.0 then you will have to do a little bit of setup.  However, with Rails it won’t be too difficult. First change the Rails gem version 5.2, run bundle install and then run 'rails app:update'. This will take you through a variety of changes so be careful to not overwrite anything in your existing app. 

Once updated you are ready to enjoy the magic of Active Storage. 

*Run a migration that builds your stroage table.  Run `rails active_storage:install` and then `rails db:migrate` to run the migrations. 

*Declare active storage in config/storage.yml 

```
 test:
 service: Disk
 root: <%= Rails.root.join("tmp/storage") %>

local:
 service: Disk
  root: <%= Rails.root.join("storage") %>
```


* Because Rails makes it easy to switch storage to a 3rd party storage like Amazon S3 you will need to make sure the "config/environments/development.rb" file is confirgured properly with 
`config.active_storage.service = :local`.  

The image file ( or any attachment file )doesn’t directly give you the ability to add image as an attribute to your class.  Instead the migration we ran earlier creates two separates tables that handle the storage and attachments. To establish this relationship all that you need to do is add the below to your model:
```
class Creator < ApplicationRecord
  has_one_attached :image
```

If you want multiple images, you will add: 
```
`has_many_attached :images`
```



Once this is done the controller needs to be updated to whitelist the image and add "with_attached_image" method to the data you are sending back as JSON. This method comes with Active Storage and is called on the instance of the class as shown below. 
```
class Api::CreatorsController < ApplicationController
...

def index
    @creators = Creator.order_by_trending
    render json: @creators.with_attached_image
  end
.....

  def creator_params
    params.require(:creator).permit(:creator_name, :image, :trending, :platform, :category, :bio)
  end
```

The serializer will have to be configured a little to properly send the image up to React to display properly in your object. 

```
class CreatorSerializer < ActiveModel::Serializer
  include Rails.application.routes.url_helpers

  attributes ..., :image, 

  def image
    return unless object.image.attached?

    object.image.blob.attributes
          .slice('filename', 'byte_size')
          .merge(url: image_url)
          .tap { |attrs| attrs['name'] = attrs.delete('filename') }
  end

  def image_url
    url_for(object.image)
  end

end
```
The image method will return a formatted object if an image is attached to an instance of a class, in this case, to the creator instance.  Within this object is the url which will be important later on to display the image.

*Note for the urlfor mehod you must include “Rails.application.routes.urlhelpers”.
 If you get an error: ArgumentError (Missing host to link to! Please provide the :host parameter, set defaulturloptions[:host], or set :onlypath to true)you probably don’t have correct environments configurations. Add these two lines with your host: config.actionmailer.defaulturloptions = { host: ‘localhost:3001’ } Rails.application.routes.defaulturloptions[:host] = ‘localhost:3001’ to every config/{environment}.rb file
**

**Installing DropZone**

Start off by switching back to your client directory and installing Dropzone in the command line
```
npm install –save react-dropzone

```

Below is a basic implementation of the Dropzone component used in my app. You can import the Dropzone compent and then add it to your form where needed. From the React Dropzone docs the basic functionality of the component works as follows: 

> “The useDropzone hook just binds the necessary handlers to create a drag 'n' drop zone. Use the getRootProps() fn to get the props required for drag 'n' drop and use them on any element. For click and keydown behavior, use the getInputProps()fn and use the returned props on an input.”

 Dropzone comes with a lot of customization abilities and built in functions. This example has the ability to check and make sure that only certain image files are accepted as well as tell the user when the item has been added to state.

```
class CreatorForm extends Component {
…
 <Dropzone onDrop={this.onDrop} accept="image/png, image/gif,image/jpg,image/jpeg" >

            {({getRootProps, getInputProps}) => (
              <div {...getRootProps()}>
									<input {...getInputProps()} />
								{this.state.image !== null ? "File Uploaded" :
								"Click me to upload a file!" }
              </div>
            )}
 </Dropzone>
	....
```
The next step it to handle the onDrop event listener. This function will set the state image value to the image that was just uploaded using the accepted file object that Dropzone just created. The image value for state should be set to null for one image or an empty array [] for multiple images.

class CreatorForm extends Component {
…..
   onDrop = (acceptedFiles) => {
     
     this.setState({
        image: acceptedFiles[0]
     });
   }
…

**Submitting the Form and sending the data to Active Storage**

I found this part a bit tricky. After a few hours and several blog posts, I discovered that Active Storage couldn’t take the object that Dropzone formatted by just the saving the current state to a variable. Instead the variable had to be a prototype of the a FormData object. Below is a creator variable that is passed into the createCreator function.  
```
   handleSubmit = event => {
     event.preventDefault()

     const creator= new FormData();
     creator.append('[creator]creator_name', this.state.creator_name)
     creator.append('[creator]image', this.state.image)
     creator.append('[creator]platform', this.state.platform)
     creator.append('[creator]bio', this.state.bio)
     creator.append('[creator]category', this.state.category)

     this.props.createCreator(creator)
```
The Action with the Fetch Request
 Now that newly saved formData can be sent as a POST request to our API endpoint. Commented out below is very important, Active Storage won’t allow attachments to be sent with headers. To be accepted by the database, the headers must be removed.  After a POST request is sent and the JSON response comes back it’s easy to dispatch an action to add the new creator to the store with the rest of the creatures.
```
  export function createCreator(creator){
    return dispatch => {

      dispatch({ type: 'SENDING_CREATOR' });
      return fetch(`/api/creators`, {
        method: "POST",
        // removed headers to get active storage attachments to work
        // headers: {
        //   'Content-Type': 'application/json'
        // },
        body: creator
    })
        .then(response => response.json())
        .then(creator => dispatch({ type: 'CREATE_CREATOR', creator: creator }));
    };
  }
```

**Displaying the image**

The new creator has been added to the database and is also in our store. The next step is displaying the image. The image is part of the JSON creator object. You can access it just like any other data that is displayed. Below is in a child component used to display the image. Because the image is an object and you will need to access the image.url property you will need to be careful of the props lifecycle and set a default value to image to an empty {} if needed.

```
<img className="background-image" src={this.props.image} alt="background" />
```

That’ll do it. With Active Storage and React Dropzone it’s easy to manage and display user images. 

From more information please check out [The Rails/ActiveStorage Documentation](https://guides.rubyonrails.org/active_storage_overview.html) ,  [The React DropZone](https://react-dropzone.netlify.com/) documentation and this awesome [blog](https://www.nopio.com/blog/upload-files-with-rails-active-storage/).

