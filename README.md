# ratesdotca-backend (BACK-END DEV ASSIGNMENT)


## Question 1
Please show what MYSQL tables and what classes you would create to accommodate for the
following 2 page form: (You can use any language, or just pseudocode for the classes)


**First Page**

First Name: [ ]

Last Name: [ ]

Phone: [ ]


**Second Page**

Vehicle 1 Vehicle 2 Vehicle 3 Vehicle 4

Year: [ ] Year: [ ] Year: [ ] Year: [ ]

Make: [ ] Make: [ ] Make: [ ] Make: [ ]

Model: [ ] Model: [ ] Model: [ ] Model: [ ] 



### Answer (Tables)

The following tables will be created:

`customer` - for storing customer data from page 1
fields: 
```
id bigint(20) unsigned not null primary key auto_increment,
firstname varchar(128) not null,
lastname varchar(128) not null,
phone varchar(32) not null,
created_at timestamp default current_timestamp(),
updated_at timestamp default current_timestamp() on update current_timestamp
```

`make` - for storing brand of the car/vehicle
fields:
```
id smallint(5) unsigned not null primary key auto_increment,
make_name varchar(128) not null
```

`model` - for storing the model of a particular make of the car/vehicle
fields:
```
id int(11) unsigned not null primary key auto_increment,
make_id smallint(5) not null,
model_name varchar(128) not null,
year smallint(4) not null,
foreign key make_id references make(id)
```

`vehicle` - for storing vehicle data associated to the customer
```
id bigint(20) unsigned not null primary key auto_increment,
model_id int(11) unsigned not null,
customer_id bigint(20) unsigned not null,
created_at timestamp default current_timestamp(),
updated_at timestamp default current_timestamp() on update current_timestamp,
foreign key (customer_id) references customer(id),
foreign key (model_id) references model(id)
```


### Answer - Model Classes

Following model classes will be created:

* `Customer` - For the customer data form in page 1.
* `Make` - For the make/brand names to store at the backend.
* `Model` - For the model names to store at the backend.
* `Vehicle` - For the vehicle data form IDs in page 2. 

## Question 2

Now that you have the tables and class models created, please explain what MVC classes you would create to make this form, and save the form data to the database. (You can use any MVC frame work in any language, or just pseudocode to demonstrate the MVC pattern)  

### Classes / templates to use when creating the page 1 form

#### Controller
`CustomerController` - class will have an `add` method to be used as a controller method for displaying the page 1 of the form
```
class CustomerController
{
    public function add()
    {
        return view('customer_form');
    }
 }
 ```
#### View template
`customer_form.blade.php` - template file where the user can enter their page 1 details. Form method is POST and will have the following inputs:

First Name (firstname)
Last Name (lastname)
Phone  number (phone)

#### Classes to use when saving the page 1 form data

`CustomerController` - class will have a `save` method where customer data will be saved
```
class CustomerController
{
    // ...add() codes here
    
    public function save(Request $request)
    {
         // Get form data, save into $data
         // instantiate an instance of Customer model class $customer
         // Set customer data
         // Save customer data through a $customer->save() method
         // Redirect to page 2 of the form
     }
 }
```

`Customer` - model class will have a `save` method where customer data will be inserted/updated to the database

```
class Customer
{
    // ...Rest of the setter, getter, etc codes here...
    public function save()
    {
        // Initialize save insert/update to the database here using the object's properties as its data
    }
}
```
 
 ### Classes/templates to use when creating the page 2 form
 #### Controller
 `VehicleController` - this will have an `add` method for displaying the 2nd page's form. It will also have a `getModels` controller method that will load the models that fall under a certain make thru an Ajax call.
 
 ```
 class VehicleController
 {
     public function add()
     {
         $vars = [
             'makes' => Make::getList(),
         ];
         // Return the template and use the variables $vars for the dropdown choices
         return view('vehicle_form', $vars);
     }
     
     public function getModels()
     {
         // Get year and make data, if available
         // Retrieve the models data
         $models = Model::getList($year, $makeId);
         // Return data in JSON format
         return json_encode($models);
      }
 }
 ```
 
 #### View template
`vehicle_form.blade.php` - template file where the user can enter the vehicle details in page 2. Form method is POST and will have the following inputs (4 times or more):

* Year (year) - text input 
* Make (make_id) - dropdown with choices coming from $makes, which came from Make::getList()
* Model (model_id) - choices coming from an AJAX call to `VehicleController::getModels()` method that executes after year and make data are supplied


 #### Model classes
 `Customer` - for retrieving the customer model associated to the Vehicle for user's reference, just in case
 ```
 class Customer
 {
     // ...Setter, getter methods here
     public function getCustomerById($id)
     {
         // Retrieve a customer model by $id into $customer
         // Return $customer object found
     }
 }
 ```
 
 
 `Make` - for retrieving a list of makes for a vehicle
 ```
 class Make
 {
     // Setter, getter methods above...
     public static function getList()
     {
         // Query the database to get a list of available makes
         // Store makes into a $makes array keyed by make ID with the brand name as the value
         // $makes[$makeId] = $name
         // Return the $makes array
     }
 }
 ```
 
 `Model` - for retrieving a list of models once the Make ID `$makeId` is specified
```
 class Model
 {
     // Setter, getter methods above...
     public static function getList($year = null, $makeId = null)
     {
         // Query the database to get a list of available makes
         // Use the $year and/or make ID parameters, if supplied, for the WHERE clause
         // Store makes into a $models array keyed by model ID with the brand name as the value
         // $models[$modelId] = $name
         // Return the $models array
     }
 }
 ```
 
#### Classes to use when saving the page 2 vehicle data 

`VehicleController` - class will have a `save` method where vehicle data will be saved
```
class VehicleController
{
    // ...add() codes here
    
    public function save(Request $request)
    {
         // Get form data, save into $data
         // instantiate an instance of Vehicle model class $vehicle
         // Set vehicle data (year, model_id)
         // Save customer data through a $vehicle->save() method
         // Redirect to page 2 of the form
     }
 }
```

`Vehicle` - model class will have a `save` method where vehicle data will be inserted/updated to the database

```
class Vehicle
{
    // ...Rest of the setter, getter, etc codes here...
    public function save()
    {
        // Initialize save insert/update to the database here using the object's properties as its data
    }
}
```


## Question 3

Now let’s assume you have your program online, then your manager come and tell you to add a parameter to Vehicle, let say the “color” of the car.
Then day 2 he wants to add “length” of the car,
Then day 3 he wants to add the “trim” of the car,
Then day 4 he wants to add tire “size” of the car,

He continues to make requests every day to add more fields to the vehicles form.
Is there any way to change your database design so that for any future requests, you do not
have to change database but still be able to add more fields and data for each form?
(No change to table structure, but still be able to add data to the tables of your new design)
And how would your MVC classes look like now that you have this new design?

### Answer

If the manager requests for an additional field frequently, we should add tables based on the Entity-Attribute-Value (EAV) pattern. This will allow the business to add fields on the fly. 

Below is a list of additional tables and MVC classes to be added.

### Additional tables 

`entity_attributes` - This table will store data about the additional attributes that could be added (attribute's code, label, data type, etc)
fields:
```
id smallint(5) not null primary key auto_increment,
code varchar(128) not null,
label varchar(255) not null,
data_type varchar(32) not null,
is_required smallint(2) not null default 0,
is_unique smallint(2) not null default 0
```

`customer_entity_<data-type>` tables will be created. Each will have a `value` column that will have a data type of varchar/int/decimal/text/datetime depending on the table's name. E.g. 

* `customer_entity_varchar`
fields:
```
id bigint(20) not null primary key auto_increment,
customer_id bigint(20) unsigned not null,
attribute_id smallint(5) not null,
value varchar(255)
```

* `customer_entity_int` - value` will have a data type of `int(11)`
* `customer_entity_decimal` - `value` will be of type `decimal(12, 4)`
* `customer_entity_text` - Field `value` will be of type `text` 
* `customer_entity_datetime` - `value` will be of type `datetime`

Same will be done for additional vehicle field requests. `vehicle_entity_<data-type>` tables will be created. Field `value` will have a data type depending on table's name. E.g. 

* `vehicle_entity_varchar` will have a `value` column of type `varchar(255)`

```
id bigint(20) not null primary key auto_increment,
vehicle_id bigint(20) unsigned not null,
attribute_id smallint(5) not null,
value varchar(255)
```

* `vehicle_entity_int` - Field `value` will be of type int(11)
* `vehicle_entity_decimal` - `value` will be of type `decimal(12, 4)`
* `vehicle_entity_text` - `value` will be of type of `text`
* `vehicle_entity_datetime` - `value` will be of type `datetime`


### Additional model/s

`EntityAttribute` - For the additional columns/attributes that could be added to the customer or vehicle. This will be mapped to the `entity_attributes` table. 
```
class EntityAttribute
{
    private $id;
    private $code;
    // and the rest of the fields $label, $dataType,  $isRequired, $isUnique
    public function setId($id)
    {
        $this->id = $id;
    }
    public function getId()
    {
        return $this->id;
    }
    // .. and the rest of the setter and getter methods for $code, $label, $dataType, $isRequired, $isUnique
    public function save()
    {
        // Save requested additional fields to the entity_attributes table
    }
}
```

`CustomerEntityRepository` - For saving / retrieving data towards the `customer_entity_<data-type>` tables.
```
class CustomerEntityRepository
{
    public static function getAttributes($customerId)
    {
        // Query customer_entity_<data-type> tables
        // Store results into the $attributes property as array
        // Keyed by attribute_id:
        // $attributes[$attribute_id] = $customerEntityAttributeRow
        // Return $attributes
    }
    
    public static function saveAttributes($customerId, array $attributes)
    {
        // Loop through each attribute in $attributes array
        // Save each attribute to their respective tables
    }
}
```

`VehicleEntityRepository` - For saving / retrieving data towards the `vehicle_entity_<data-type>` tables.
```
class VehicleEntityRepository
{
    public static function getAttributes($vehicleId)
    {
        // Query customer_entity_<data-type> tables
        // Store results into the $attributes property as array
        // Keyed by attribute_id:
        // $attributes[$attribute_id] = $vehicleEntityAttributeRow
        // Return $attributes
    }
    
    public static function saveAttributes($vehicleId, array $attributes)
    {
        // Loop through each attribute in $attributes array
        // Save each attribute to their respective tables
    }
}
```

### Model updates

`Customer` - customer model class will call the `CustomerEntityRepository::getAttributes` and `CustomerEntityRepository::saveAttributes` methods to retrieve / save data
```
class Customer
{
     // Add the $attributes array property below the initially declared properties
     private $attributes;
    
     // ...rest of methods here
    
    public function getCustomerById($id)
    {
        // Retrieve a customer model by $id into $customer
        // Call the CustomerEntityRepository::getAttributes method to get other customer data
        $this->attributes = CustomerEntityRepository::getAttributes($id);
        // Return $customer object found
    }
    
    public function save()
    {
        // Initialize save insert/update to the database here using the object's properties as its data
        // Save additional data thru CustomerEntityRepository::saveAttributes()
        CustomerEntityRepository::saveAttributes($customerId, $this->attributes);
        return $customer;
    }
}
```
        
`Vehicle` - vehicle model class will call the `VehicleEntityRepository::getAttributes` and `VehicleEntityRepository::saveAttributes` methods to retrieve / save data
```
class Vehicle
{
     // Add the $attributes array property below the initially declared properties
     private $attributes;

     // ...rest of methods here

    public function getVehicleById($id)
    {
        // Retrieve a vehicle model by $id into $vehicle
        // Call the VehicleEntityRepository::getAttributes method to get other vehicle data
        $this->attributes = VehicleEntityRepository::getAttributes($id);
        // Return $vehicle object found
    }

    public function save()
    {
        // Initialize save insert/update to the database here using the object's properties as its data
        // Save additional data thru VehicleEntityRepository::saveAttributes()
        VehicleEntityRepository::saveAttributes($vehicleId, $this->attributes);
        return $vehicle;
    }
}
```

### View template updates

The following templates will be updated such that the rest of the form inputs with a name of `attributes[<attribute_id]`. These inputs will be put under the fields that were initially created for the customer (which is the `phone` field) and vehicle (`model` field).

customer_form.blade.php
* First name input
* Last name input
* Phone input
* attributes[attribute_id] input, etc.

vehicle_form.blade.php
* Year input
* Make dropdown
* Model dropdown
* attributes[attribute_id] input, etc.

