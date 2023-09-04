Ideally for java we would like a syntax which reads well and allows single statement calls, but is still mostly type safe. 
(When trying to represent a generalized syntax, java frameworks are typcially rather clunky)
One possible approach is usage of auto generated 'Shape' objects which have attributes/methods which follow the object model.
This would allow something like (given appointment and customer Shape objects)

```java
AppointmentService.getAppointments(builder -> builder
   .params.locationId(locationId)
   .params.fromDate(fromDate)
   .filter(
      appointment.status.in("SCHEDULED", "CONFIRMED") // only SCHEDULED or CONFIRMED
   )
   .append(
      Append.as("customer", CustomerService.givenAppointments.getCustomer()), // append customer
      Append.as("associate", AssociateService.givenAppointments.getAssociate()) // append associate
   )
   .sort(
      appointment.scheduledDate.asc, // scheduledDate
      Sort.on("customer",
         customer.lastName.asc, // then customer name
         customer.firstName.asc
      ),
      appointment.appointmentId.asc
   )
   .paginate(10) // 10 at a time
);
```

Needs further thought.
