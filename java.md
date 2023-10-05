Ideally for java we would like a syntax which reads well and allows single statement calls, but is still mostly type safe. 
(When trying to represent a generalized syntax, java frameworks are typcially rather clunky)
One possible approach is usage of auto generated attribute data and helper funtions which follow the object model.
This would allow something like

```java
Page<Appointment> appointments = AppointmentService.getAppointments(AppointmentService.getAppointments
   .params.locationId(locationId)
   .params.fromDate(fromDate)
   .filter(
      Appointment.shape.status.in("SCHEDULED", "CONFIRMED") // only SCHEDULED or CONFIRMED
   )
   .append(
      Customer.as("customer", CustomerService.givenAppointments.getCustomer()), // append customer
      Associate.as("associate", AssociateService.givenAppointments.getAssociate()) // append associate
   )
   .sort(
      Appointment.shape.scheduledDate.asc, // scheduledDate
      Customer.on("customer",
         Customer.shape.lastName.ignoreCase.asc, // then customer name
         Customer.shape.firstName.ignoreCase.asc
      ),
      Appointment.shape.appointmentId.asc
   )
   .paginate(10) // 10 at a time
);
```

Needs further thought.
