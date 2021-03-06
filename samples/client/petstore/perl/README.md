# NAME

WWW::SwaggerClient::Role - a Moose role for the Swagger Petstore 

## Swagger Petstore version: 1.0.0

# VERSION

Automatically generated by the Perl Swagger Codegen project: 

- Build date: 2016-03-13T22:43:11.863+08:00
- Build package: class io.swagger.codegen.languages.PerlClientCodegen
- Codegen version: 

## A note on Moose

This role is the only component of the library that uses Moose. See 
WWW::SwaggerClient::ApiFactory for non-Moosey usage. 

# SYNOPSIS

The Perl Swagger Codegen project builds a library of Perl modules to interact with 
a web service defined by a OpenAPI Specification. See below for how to build the 
library.

This module provides an interface to the generated library. All the classes, 
objects, and methods (well, not quite \*all\*, see below) are flattened into this 
role. 

        package MyApp;
        use Moose;
        with 'WWW::SwaggerClient::Role';
        
        package main;
        
        my $api = MyApp->new({ tokens => $tokens });
        
        my $pet = $api->get_pet_by_id(pet_id => $pet_id);
        

## Structure of the library

The library consists of a set of API classes, one for each endpoint. These APIs
implement the method calls available on each endpoint. 

Additionally, there is a set of "object" classes, which represent the objects 
returned by and sent to the methods on the endpoints. 

An API factory class is provided, which builds instances of each endpoint API. 

This Moose role flattens all the methods from the endpoint APIs onto the consuming 
class. It also provides methods to retrieve the endpoint API objects, and the API 
factory object, should you need it. 

For documentation of all these methods, see AUTOMATIC DOCUMENTATION below.

## Configuring authentication

In the normal case, the OpenAPI Spec will describe what parameters are
required and where to put them. You just need to supply the tokens.

    my $tokens = {
        # basic
        username => $username,
        password => $password,
        
        # oauth
        access_token => $oauth_token,
        
        # keys
        $some_key => { token => $token,
                       prefix => $prefix, 
                       in => $in,             # 'head||query',     
                       },
                       
        $another => { token => $token,
                      prefix => $prefix, 
                      in => $in,              # 'head||query',      
                      },                   
        ...,
        
        };
        
        my $api = MyApp->new({ tokens => $tokens });

Note these are all optional, as are `prefix` and `in`, and depend on the API
you are accessing. Usually `prefix` and `in` will be determined by the code generator from
the spec and you will not need to set them at run time. If not, `in` will
default to 'head' and `prefix` to the empty string. 

The tokens will be placed in the `WWW::SwaggerClient::Configuration` namespace
as follows, but you don't need to know about this. 

- `$WWW::SwaggerClient::Configuration::username`

    String. The username for basic auth.

- `$WWW::SwaggerClient::Configuration::password`

    String. The password for basic auth.

- `$WWW::SwaggerClient::Configuration::api_key`

    Hashref. Keyed on the name of each key (there can be multiple tokens).

            $WWW::SwaggerClient::Configuration::api_key = {
                    secretKey => 'aaaabbbbccccdddd',
                    anotherKey => '1111222233334444',
                    };

- `$WWW::SwaggerClient::Configuration::api_key_prefix`

    Hashref. Keyed on the name of each key (there can be multiple tokens). Note not
    all api keys require a prefix.

            $WWW::SwaggerClient::Configuration::api_key_prefix = {
                    secretKey => 'string',
                    anotherKey => 'same or some other string',
                    };

- `$WWW::SwaggerClient::Configuration::access_token`

    String. The OAuth access token. 

# METHODS

## `base_url`

The generated code has the `base_url` already set as a default value. This method 
returns (and optionally sets, but only if the API client has not been 
created yet) the current value of `base_url`.

## `api_factory`

Returns an API factory object. You probably won't need to call this directly. 

        $self->api_factory('Pet'); # returns a WWW::SwaggerClient::PetApi instance
        
        $self->pet_api;            # the same

# MISSING METHODS

Most of the methods on the API are delegated to individual endpoint API objects
(e.g. Pet API, Store API, User API etc). Where different endpoint APIs use the
same method name (e.g. `new()`), these methods can't be delegated. So you need
to call `$api->pet_api->new()`.

In principle, every API is susceptible to the presence of a few, random, undelegatable 
method names. In practice, because of the way method names are constructed, it's 
unlikely in general that any methods will be undelegatable, except for: 

        new()
        class_documentation()
        method_documentation()

To call these methods, you need to get a handle on the relevant object, either
by calling `$api->foo_api` or by retrieving an object, e.g.
`$api->get_pet_by_id(pet_id => $pet_id)`. They are class methods, so
you could also call them on class names.

# BUILDING YOUR LIBRARY

See the homepage `https://github.com/swagger-api/swagger-codegen` for full details. 
But briefly, clone the git repository, build the codegen codebase, set up your build 
config file, then run the API build script. You will need git, Java 7 or 8 and Apache
maven 3.0.3 or better already installed.

The config file should specify the project name for the generated library: 

        {"moduleName":"WWW::MyProjectName"}

Your library files will be built under `WWW::MyProjectName`.

          $ git clone https://github.com/swagger-api/swagger-codegen.git
          $ cd swagger-codegen
          $ mvn package
          $ java -jar modules/swagger-codegen-cli/target/swagger-codegen-cli.jar generate \
    -i [URL or file path to JSON swagger API spec] \
    -l perl \
    -c /path/to/config/file.json \
    -o /path/to/output/folder

Bang, all done. Run the `autodoc` script in the `bin` directory to see the API 
you just built. 

# AUTOMATIC DOCUMENTATION

You can print out a summary of the generated API by running the included
`autodoc` script in the `bin` directory of your generated library. A few
output formats are supported:

          Usage: autodoc [OPTION]

    -w           wide format (default)
    -n           narrow format
    -p           POD format 
    -H           HTML format 
    -m           Markdown format
    -h           print this help message
    -c           your application class
    

The `-c` option allows you to load and inspect your own application. A dummy
namespace is used if you don't supply your own class.

# DOCUMENTATION FROM THE OpenAPI Spec

Additional documentation for each class and method may be provided by the Swagger 
spec. If so, this is available via the `class_documentation()` and 
`method_documentation()` methods on each generated object class, and the 
`method_documentation()` method on the endpoint API classes: 

        my $cmdoc = $api->pet_api->method_documentation->{$method_name}; 
        
        my $odoc = $api->get_pet_by_id->(pet_id => $pet_id)->class_documentation;                                  
        my $omdoc = $api->get_pet_by_id->(pet_id => $pet_id)->method_documentation->{method_name}; 
        

Each of these calls returns a hashref with various useful pieces of information.

# LOAD THE MODULES

To load the API packages:
```perl
use WWW::SwaggerClient::PetApi;
use WWW::SwaggerClient::StoreApi;
use WWW::SwaggerClient::UserApi;

```

To load the models:
```perl
use WWW::SwaggerClient::Object::Category;
use WWW::SwaggerClient::Object::InlineResponse200;
use WWW::SwaggerClient::Object::ModelReturn;
use WWW::SwaggerClient::Object::Name;
use WWW::SwaggerClient::Object::Order;
use WWW::SwaggerClient::Object::Pet;
use WWW::SwaggerClient::Object::SpecialModelName;
use WWW::SwaggerClient::Object::Tag;
use WWW::SwaggerClient::Object::User;

````

# DOCUMENTATION FOR API ENDPOINTS

All URIs are relative to *http://petstore.swagger.io/v2*

Class | Method | HTTP request | Description
------------ | ------------- | ------------- | -------------
*PetApi* | [**add_pet**](docs/PetApi.md#add_pet) | **POST** /pet | Add a new pet to the store
*PetApi* | [**add_pet_using_byte_array**](docs/PetApi.md#add_pet_using_byte_array) | **POST** /pet?testing_byte_array=true | Fake endpoint to test byte array in body parameter for adding a new pet to the store
*PetApi* | [**delete_pet**](docs/PetApi.md#delete_pet) | **DELETE** /pet/{petId} | Deletes a pet
*PetApi* | [**find_pets_by_status**](docs/PetApi.md#find_pets_by_status) | **GET** /pet/findByStatus | Finds Pets by status
*PetApi* | [**find_pets_by_tags**](docs/PetApi.md#find_pets_by_tags) | **GET** /pet/findByTags | Finds Pets by tags
*PetApi* | [**get_pet_by_id**](docs/PetApi.md#get_pet_by_id) | **GET** /pet/{petId} | Find pet by ID
*PetApi* | [**get_pet_by_id_in_object**](docs/PetApi.md#get_pet_by_id_in_object) | **GET** /pet/{petId}?response=inline_arbitrary_object | Fake endpoint to test inline arbitrary object return by &#39;Find pet by ID&#39;
*PetApi* | [**pet_pet_idtesting_byte_arraytrue_get**](docs/PetApi.md#pet_pet_idtesting_byte_arraytrue_get) | **GET** /pet/{petId}?testing_byte_array=true | Fake endpoint to test byte array return by &#39;Find pet by ID&#39;
*PetApi* | [**update_pet**](docs/PetApi.md#update_pet) | **PUT** /pet | Update an existing pet
*PetApi* | [**update_pet_with_form**](docs/PetApi.md#update_pet_with_form) | **POST** /pet/{petId} | Updates a pet in the store with form data
*PetApi* | [**upload_file**](docs/PetApi.md#upload_file) | **POST** /pet/{petId}/uploadImage | uploads an image
*StoreApi* | [**delete_order**](docs/StoreApi.md#delete_order) | **DELETE** /store/order/{orderId} | Delete purchase order by ID
*StoreApi* | [**find_orders_by_status**](docs/StoreApi.md#find_orders_by_status) | **GET** /store/findByStatus | Finds orders by status
*StoreApi* | [**get_inventory**](docs/StoreApi.md#get_inventory) | **GET** /store/inventory | Returns pet inventories by status
*StoreApi* | [**get_inventory_in_object**](docs/StoreApi.md#get_inventory_in_object) | **GET** /store/inventory?response=arbitrary_object | Fake endpoint to test arbitrary object return by &#39;Get inventory&#39;
*StoreApi* | [**get_order_by_id**](docs/StoreApi.md#get_order_by_id) | **GET** /store/order/{orderId} | Find purchase order by ID
*StoreApi* | [**place_order**](docs/StoreApi.md#place_order) | **POST** /store/order | Place an order for a pet
*UserApi* | [**create_user**](docs/UserApi.md#create_user) | **POST** /user | Create user
*UserApi* | [**create_users_with_array_input**](docs/UserApi.md#create_users_with_array_input) | **POST** /user/createWithArray | Creates list of users with given input array
*UserApi* | [**create_users_with_list_input**](docs/UserApi.md#create_users_with_list_input) | **POST** /user/createWithList | Creates list of users with given input array
*UserApi* | [**delete_user**](docs/UserApi.md#delete_user) | **DELETE** /user/{username} | Delete user
*UserApi* | [**get_user_by_name**](docs/UserApi.md#get_user_by_name) | **GET** /user/{username} | Get user by user name
*UserApi* | [**login_user**](docs/UserApi.md#login_user) | **GET** /user/login | Logs user into the system
*UserApi* | [**logout_user**](docs/UserApi.md#logout_user) | **GET** /user/logout | Logs out current logged in user session
*UserApi* | [**update_user**](docs/UserApi.md#update_user) | **PUT** /user/{username} | Updated user


# DOCUMENTATION FOR MODELS
 - [WWW::SwaggerClient::Object::Category](docs/Category.md)
 - [WWW::SwaggerClient::Object::InlineResponse200](docs/InlineResponse200.md)
 - [WWW::SwaggerClient::Object::ModelReturn](docs/ModelReturn.md)
 - [WWW::SwaggerClient::Object::Name](docs/Name.md)
 - [WWW::SwaggerClient::Object::Order](docs/Order.md)
 - [WWW::SwaggerClient::Object::Pet](docs/Pet.md)
 - [WWW::SwaggerClient::Object::SpecialModelName](docs/SpecialModelName.md)
 - [WWW::SwaggerClient::Object::Tag](docs/Tag.md)
 - [WWW::SwaggerClient::Object::User](docs/User.md)


# DOCUMENTATION FOR AUTHORIATION

## test_api_key_header

- **Type**: API key 
- **API key parameter name**: test_api_key_header
- **Location**: HTTP header

## api_key

- **Type**: API key 
- **API key parameter name**: api_key
- **Location**: HTTP header

## test_http_basic

- **Type**: HTTP basic authentication

## test_api_client_secret

- **Type**: API key 
- **API key parameter name**: x-test_api_client_secret
- **Location**: HTTP header

## test_api_client_id

- **Type**: API key 
- **API key parameter name**: x-test_api_client_id
- **Location**: HTTP header

## test_api_key_query

- **Type**: API key 
- **API key parameter name**: test_api_key_query
- **Location**: URL query string

## petstore_auth

- **Type**: OAuth
- **Flow**: implicit
- **Authorizatoin URL**: http://petstore.swagger.io/api/oauth/dialog
- **Scopes**: 
 - **write:pets**: modify pets in your account
 - **read:pets**: read your pets



