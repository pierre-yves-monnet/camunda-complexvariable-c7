# camuinda-complexvariable-c7

Demonstrate the different possiblities to handle a complex variable in Camunda 7
Data are defined under org.camunda.complexvariable.c7.data, class Customer
Process (ComplexVariables) defined a simple process which call a JavaDelegate, a Worker, a JavaDelegate and a Worker.

Each JavaDelegate and Worker manipulate 3 types of variables:
* a Customer variable, saved in JSON. All operations are corrects
* a Customer variable, saved in JAVA SERIALIZATION. The save is NOT POSSIBLE from the worker (Exception occure)
* A Customer variable, direct access. All operations are corrects.


# JavaDelegate
org.camunda.complexvariable.c7.delegate.DelegateVariables
is a Delegate component. This component manipulates 3 different Customer variables, with 3 different ways:
* Java
* JSON
* Direct access (Serialization behind the scene)

To start the Delegate, the Camunda Engine is under org.camunda.complexvariable.c7.engine. 
Start the static main under CamundaEngine

# External Worker
The external worker access the three different kind of variables.

org.camunda.complexvariable.c7.workerVariable
Start the main() in the worker. This is an external application

# Test it
Start the Camunda Engine, then start the External worker.


# Forms
How to display/edit a complex data in a form?
Three different possibility are study.
## html
Form is placed under resources/static/forms/displayCustomer.html
Form key is embedded:app:forms/displayCustomer.html

https://docs.camunda.org/manual/7.8/reference/embedded-forms/json-data/
It is not possible to link a variable by

````
                <input id="FirstName"
                       type="text"
                       cam-variable-name="JsonCustomer.firstName"
                       cam-variable-type="String"
                        />
````
So, some Javascript is mandatory.
This method works for JSON, and Direct DATA, not for a JAVA Serialized data. The REST CALL to fetch data do not deserialized
(https://forum.camunda.org/t/variablemanager-fetchvariable-return-not-serialized-value/14086)

1. Explicitly upload the variable in the page, and save it in a local variable

````
    <script cam-script type="text/form-script">

   camForm.on('form-loaded', function() {
     // fetch the variable named 'customerData'
     camForm.variableManager.fetchVariable('JsonCustomer');
   });

   camForm.on('variables-fetched', function() {
     // after the variables are fetched, bind the value of customerData to a angular
     // scope value such that the form can work on it
     $scope.localJsonCustomer = camForm.variableManager.variable('JsonCustomer').value;
    });
````

2. Use the new local variable in your page

Note: you have to use ng-model here. 
````
    <input id="FirstName"
           type="text"
           ng-model="localJsonCustomer.firstName"
            />
````

3.at submit, save the content of the variable in the Camunda scope

````
camForm.on('submit', function() {
   camForm.variableManager.variable('JsonCustomer').value=$scope.localJsonCustomer;
    });
````    
## Embedded form
ID is       JsonCustomer.firstName
value is    ${JsonCustomer.firstName}

## Camunda form
Form is placed under resources/static/forms. Name is complexVariable.form
Type: Embedded form
FormKey: camunda-forms:app:forms/complexeVariable.form

ATTENTION: you must not choose the "Embedded or task form" type, and not "Camunda-form". 
If you choose Camunda-form, you have to setup a formId, and give this formId as the reference. 
Then, the form must be deployed with the process via the REST API or the modeler, and not in a static way.

With a Camunda form, this is not possible to reference a variable like "JsonCustomer.firstName". 
For each information you want to display, you have to create an input (like JsonCustomerFirstName) and populate the 
input with the expression ${JsonCustomer.firstName}
On the output, a listener has to be created. Indeed, when "JsonCustomer.firstName" is set as the VariableName, 
Camunda Engine creates a variable name "JsonCustomer.firstName" and do not populate attribut "firstName" in the "JsonCustomer" variable.

# Different program

## org.camunda.complexvariable.c7.engine.CamundaEngine
Start a C7 engine, with CamundaPostConstruct to create a process instance

## org.camunda.worker.SimpleWorker
A simple external worker

