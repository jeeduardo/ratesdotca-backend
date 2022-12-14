# ratesdotca-backend


## Question 1
For page 1 customer form and page 2 vehicle form: 


### Answer - Tables

`customer` - for storing customer data from page 1
fields: 
```
id bigint(20) unsigned not null primary key auto_increment,
firstname varchar(128) not null,
lastname varchar(128) not null,
phone varchar(32) not null
```

`vehicle` - for storing vehicle data from page 2
fields: 
```
id bigint(20) unsigned not null primary key auto_increment,
year smallint(4) not null,
make varchar(128) not null,
model varchar(255) not null
```

`customer_vehicle` - for mapping relationship between customer and vehicle (just in case manager / administrator decides to view a report about a certain customer or most popular vehicles)
fields: 
```
id bigint(20) unsigned not null primary key auto_increment,
customer_id bigint(20) unsigned not null,
vehicle_id bigint(20) unsigned not null,
foreign key (customer_id) references customer(id),
foreign key (vehicle_id) references vehicle(id),
```


### Answer - Model Classes

`Customer` - For the customer data form in page 1
`Vehicle` - For the vehicle data form in page 2. I intend to put the relationship of the customer and vehicle to a separate table and class just in case a manager/administrator wants to lookup the most popular vehicles, etc. 
`CustomerVehicle` - For the customer's transaction with a vehicle

## Question 2

2. Now that you have the tables and class models created, please explain what MVC classes you would create to make this form, and save the form data to the database. (You can use any MVC frame work in any language, or just pseudocode to demonstrate the MVC pattern)  


Model classes

`Customer.php`
```
class Customer
{
    private $id;

    private $firstname;

    private $lastname;

    private $phone;

    public function setId($id)
    {
        $this->id = $id;
    }

    public function getId()
    {
        return $this->id;
    }
    
    public function setFirstname($firstname)
    {
        $this->firstname = $firstname;
    }

    public function getFirstname()
    {
        return $this->firstname;
    }

    // ...Same goes for make and model, both fields will have setter and getter methods
    // public function setLastname($lastname)
    // public function getLastname()
    // public function setPhone($phone)
    // public function getPhone()
    
}
```

`Vehicle.php`

```
class Vehicle
{
    private $id;

    private $year;

    private $make;

    private $model;

    public function setId($id)
    {
        $this->id = $id;
    }

    public function getId()
    {
        return $this->id;
    }
    
    public function setYear($year)
    {
        $this->year = $year;
    }

    public function getYear()
    {
        return $this->year;
    }
    
    // ...Same goes for make and model, both fields will have setter and getter methods
    
}
```

`CustomerVehicle.php`

```
class CustomerVehicle
{
    private $id;

    private $customerId;

    private $vehicleId;

    private $customerRepository;

    private $vehicleRepository;

    public function __construct(
        CustomerRepository $customerRepository,
        VehicleRepository $vehicleRepository
    ) {
        $this->customerRepository = $customerRepository;
        $this->vehicleRepository = $vehicleRepository;
    }

    public function setId($id)
    {
        $this->id = $id;
    }

    public function getId()
    {
        return $this->id;
    }

    // ...Same goes for customer_id and vehicle_id, both will have setter and getter methods
    // public function setCustomerId($customerId)
    // public function getCustomerId()
    // public function setVehicleId($vehicleId)
    // public function getVehicleId()

    // There'll be methods to get corresponding customer and vehicle model
    public function getCustomer()
    {
        $customer = $this->customerRepository->getById($this->customerId);
        return $customer;
    }

    public function getVehicle()
    {
        $vehicle = $this->vehicleRepository->getById($this->vehicleId);
        return $vehicle;
    }
}
```


Controller classes - 

```
class CustomerController
{
    /**
     * Controller to show page 1 of form
     * GET /customer
     */
    public function show()
    {
        return view('customer.form');
    }

    /**
      *
    public function save(
        Request $request, 
        CustomerRepository $customerRepository
    ) {
        $data = $request->all();

        $customer = $customerRepository->save($data);

        if ($customer->getId()) {
            $customerId = $customer->getId();
            return redirect('/customer/'.$customerId.'/vehicle');
        }

        return redirect('/customer/error');
    }
}
```

```
class VehicleController
{
    /**
     * Controller to show page 2 of form
     * GET /customer
     */
    public function show()
    {
        return view('vehicle.form');
    }

    /**
     * Controller to save page 2 customer data
     * @param Request $request
     * @param CustomerRepository $customerRepository
     * @return redirect
     */
    public function save(
        Request $request,
        VehicleRepository $vehicleRepository
    ) {
        $data = $request->all();

        $customer = $vehicleRepository->save($data);

        if ($customer->getId()) {
            $customerId = $customer->getId();
            return redirect('/vehicle/success');
        }

        return redirect('/vehicle/error');
    }

}
```

View templates:

`views/customer/form.blade.php` (e.g. for Laravel) - template file for the 1st form where user can enter their details
`views/vehicle/form.blade.php` - template file for the 2nd form where user can enter the vehicles they prefer

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

### ANSWER

If the manager requests for an additional field frequently/daily, I think I would add tables based on the Entity-Attribute-Value pattern. Magento does this to allow the business to add additional fields on the fly. Below is a list of additional tables and MVC classes that will be added.

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

I will create `customer_entity_<data-type>` tables where the `value` column will have a data type of varchar/int/decimal/text/datetime. E.g. 

`customer_entity_varchar`
fields:
```
id bigint(20) not null primary key auto_increment,
customer_id bigint(20) unsigned not null,
attribute_id smallint(5) not null,
value varchar(255)
```

`customer_entity_int` - Same as customer_entity_varchar except that the `value` has a data type of `int(11)`
`customer_entity_decimal` - Field `value` will have a data type of `decimal(12, 4)`
`customer_entity_text` - Field `value` will have a data type of `text` 
`customer_entity_datetime` - Field `value` will have a data type of `datetime`

Same thing that I will do for vehicle data (`vehicle_entity_<data-type>` tables):

`vehicle_entity_varchar`

fields:
```
id bigint(20) not null primary key auto_increment,
vehicle_id bigint(20) unsigned not null,
attribute_id smallint(5) not null,
value varchar(255)
```

`vehicle_entity_int` - Same structure as vehicle_entity_varchar except that the `value` will have a data type of int(11)
`vehicle_entity_decimal` - Field `value` will have a data type of `decimal(12, 4)`
`vehicle_entity_text` - Field `value` will have a data type of `text`
`vehicle_entity_datetime` - Field `value` will have a data type of `datetime`



### Additional model/s

`EntityAttribute` - For the additional columns/attributes that could be added to the customer or vehicle

```
class EntityAttribute
{
    /**
     * @var int
     */
    private $id;

    /**
     * @var string
     */
    private $code;

    // And the rest of the fields e.g. private $label; etc...

    public function setId($id)
    {
        $this->id = $id;
    }

    /**
     * @return int
     */
    public function getId()
    {
        return $this->id;
    }

    // Rest of the fields will have their setter and getter methods also e.g.
    // public function setCode($code)
    // public function getCode()

}
```

`CustomerAttribute` - For the additional fields that will be added for the Customer

```
class CustomerAttribute
{
    /**
     * @var array
     */
    private $values;

    /**
     * Hydrate the object with attribute values coming from customer_entity_<data-type> tables
     * @param $customerId
     */
    public static function load($customerId)
    {
        // Query the customer_entity_<data-type> tables that contain data about the customer with ID $customerId
        // Store them into a $values array
        // Each element will have a key of $attributeId with the value of $value
        // $this->values[$attributeId] = $value;
        $customerAttribute = new static;
        // Assign values thru $customerAttribute->setValues($values) then return the $customerAttribute object
        return $customerAttribute;
    }

    /**
     * Set an array of values
     * @param $values
     */
    public function setValues($values)
    {
        $this->values = $values;
    }

    /**
     * Return an array of attribute values $values
     * Each element is keyed by attribute ID with its value
     * $values[$attributeId] = $value
     * @return array
     */
    public function getValues()
    {
        return $this->values;
    }
}
```

`VehicleAttribute` - For the additional fields that will be added for the Vehicle

```
class VehicleAttribute
{
    /**
     * @var array
     */
    private $values;

    /**
     * Hydrate the object with attribute values coming from customer_entity_<data-type> tables
     * @param $customerId
     */
    public function load($vehicleId)
    {
        // Store them into a $values array
        // Each element will have a key of $attributeId with the value of $value
        // $this->values[$attributeId] = $value;
        $vehicleAttribute = new static;
        // Assign values thru $vehicleAttribute->setValues($values) then return the $vehicleAttribute object
        return $vehicleAttribute;
    }

    /**
     * Return an array of attribute values $values
     * Each element is keyed by attribute ID with its value
     * $values[$attributeId] = $value
     * @return array
     */
    public function getValues()
    {
        return $this->values;
    }
}
```

### Model updates

`Customer` - customer model will have the following updates

```
class Customer
{
    // ...Rest of the existing member declaration code here

    /**
     * @var array
     */
    private $customerAttributeValues;

    // ... Rest of the existing code here
    public function getCustomerAttributeValues()
    {
        $this->customerAttributes = CustomerAttribute::load($this->id);
        return $this->customerAttribute->getValues();
    }
}
```

`Vehicle` - vehicle model will have the following updates
```
class Vehicle
{
    // ...Rest of the existing member declaration code here

    /**
     * @var array
     */
    private $vehicleAttributeValues;

    // ... Rest of the existing code here
    public function getVehicleAttributeValues()
    {
        $this->vehicleAttributes = VehicleAttribute::load($this->id);
        return $this->vehicleAttribute->getValues();
    }
}
```


### Template updates

The following templates will be updated such that the rest of the form inputs with a name of `attributes[<attribute_id]`. These inputs will be put under the fields that were initially created for the customer (which is the `phone` field) and vehicle (`model` field)
`views/customer/form.blade.php`
`views/vehicle/form.blade.php`

### Controller updates
