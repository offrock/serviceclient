# Service Client Example 
The Service Client is a generalized framework that provides client side service classes that encapsulate and expose remote interactions as simple functions. The following provides a brief introduction by example. 

## Existing Microservices
As a reference example for discussion, let’s imagine we have three existings microservices providing appointments, customers, and sales associates.  

### Appointments
An appointment service which stores basic appointment objects.

```javascript
Appointment: {
   appointmentId,
   locationId,
   scheduledDate, // ISO date time
   status, // SCHEDULED | CONFIRMED | CANCELED
   customerId,
   associateId
}
```

And includes a simple REST endpoint to paginate appointments for a location by scheduled date.

```bash
GET /locations/{locationId}/appointments?
   fromDate={fromDate}
   &toDate={toDate}
   &sort={sort}
   &pageSize={pageSize}
   &page={page}
```

### Customers
A customer service which stores customer objects.

```javascript
Customer: {
   customerId,
   firstName,
   lastName
}
```

And includes an endpoint to get a customer by customerId.

```bash
GET /customers/{customerId}
```

### Associates
And finally a sales associate service which stores associate objects.

```javascript
Associate: {
   associateId,
   locationId,
   firstName,
   lastName
}
```

And includes an endpoint to get all associates for a location.

```bash
GET /locations/{locationId}/associates
```

### Full Model
The full object model can be represented with the following diagram.

<img src="/images/fullmodel.png" width="520">

## Basic Usage
We could use these endpoints directly, but what if instead we could simply import a set of service classes and make simple function calls.

```javascript
import {Services} from 'service-client';

// choose service
const AppointmentService = Services.appointments.v2.AppointmentService;

// get all appointments for a given location and day
let appointments = await AppointmentService.getAppointments({
   locationId: 2899,
   fromDate: '2021-08-22T05:00:00.000Z',
   toDate: '2021-08-23T05:00:00.000Z'
});
```

This demonstrates the simplicity of exposing interactions as functions, but now that we have full control over the interaction from within the client, what else can we do?

## A Realistic Use Case
For something more realistic, let's say we need to render an infinite scroll of upcoming appointments for a location with customer name and sales associate name, filtered by status, sorted by appointment scheduled date and customer name.

<img src="/images/appointments.png" width="860">

We could of course make a series of api calls from the browser and work all this out, or we could make a new backend endpoint to do the same, but what if instead we could make the following request using a generalized syntax for filtering, appending related data, sorting, and pagination.

```javascript
let page = await AppointmentService.getAppointments({
   locationId,
   fromDate: moment().tz(timezone).startOf('day'), // from today
   $filter: {
      status: {$in: ['SCHEDULED', 'CONFIRMED']} // only SCHEDULED or CONFIRMED
   },
   $append: {
      customer: CustomerService.givenAppointments.getCustomer(), // append customer
      associate: AssociateService.givenAppointments.getAssociate() // append associate
   },
   $sort: [ 
      {scheduledDate: '$asc'}, // scheduledDate
      {customer: {lastName: '$asc'}}, // then customer name
      {customer: {firstName: '$asc'}},
      {appointmentId: '$asc'}
   ],
   $paginate: {$limit: 10} // 10 at a time
});

// get appointment array
let appointments = page.data();

while(page.hasMore()) {
   // get next page
   page =  await AppointmentService.getAppointments(page.next());
   appointments = page.data();
}
```

Consider what the Service Client is doing for us in this scenario

* Simplified interaction - REST/HTTP semantics and syntax (urls, headers, body, response codes, etc.) are replaced with simple function calls.
* Single request - The full request is transparently serialized (potentially batched with other requests) and sent to a Service Client running in the backend for more efficient execution. 
* Parameter transformation - The fromDate parameter accepts various types that are transparently transformed into the required string format. Parameter names can also be aliased for naming consistency across services.
* Filter - Since the appointment service endpoint does not currently support search by status, appointments are filtered using the Service Client’s $filter after effect which provides generalized object filtering.
* Append related entities - The customer and sales associate are appended to each appointment. The Service Client can take advantage of batch endpoints where available. Attribute level projections are also supported.
* Dedup and cache - Duplicate customers and sales associates can be transparently deduped over the wire. Since the number of sales associates at a given location is relatively small, sales associates could be configured to cache and append client side. 
* Cross service sort - Since the sort involves both appointment and customer attributes, the Service Client must employ a look ahead algorithm with in-memory sorting. Scenarios such as this are one reason why the Service Client provides its own sorting syntax.
* Pagination - Since appointments are filtered by the Service Client, it must extend forward and maintain a second hidden offset. Scenarios such as this are one reason why the Service Client provides its own pagination syntax.
* Housekeeping - Various concerns such as configuration, versioning, authentication, logging, etc., can all be centralized within the Service Client.
* Testing - Local reference implementation with test data sets which can be used for development and unit testing.

It should be noted that all functionally (beyond an adapter function for each endpoint) is generalized and can be applied to any scenario without any use case specific development.

This example demonstrates only a small fraction of functionality that can be provided by encapsulating remote interactions within a single client library. Solutions such as this are easy to use and can dramatically increase developer productivity and application performance with little or no imposition on existing infrastructure.

Shane Taylor \
925 785-2722 \
shane.taylor@gmail.com

https://github.com/offrock/serviceclient#readme
