# **Concierge API**

Through the Shortcut Concierge API, you are able to view available services, find Shortcut Pros, and book appointments. All API calls are accomplished by making `POST` requests to:
* https://<span></span>api.shortcutpros.com/**_function_name_** for the production environment
* https://<span></span>api-dev.shortcutpros.com/**_function_name_** for the sandbox environment

To get in touch with Shortcut and request an API Key, please reach out to support@getshortcut.co.

### Functions

[pullConciergeServices](#pullConciergeServices)

[coordinatesForAddress_Concierge](#coordinatesForAddress_Concierge)

[pullApptSlots](#pullApptSlots)

[lockApptSlot](#lockApptSlot)

[bookApptSlot](#bookApptSlot)

[cancelApptSlot](#cancelApptSlot)

[pullUpcomingAppts_Concierge](#pullUpcomingAppts_Concierge)

[pullPastAppts_Concierge](#pullPastAppts_Concierge)

### Documentation

***

#### pullConciergeServices
View all available services. Some services are `addOnOnly`, which means that they cannot be ordered without ordering a non-`addOnOnly` service along with it.

Parameters:
* _apiKey_ (string)

Returns: Array of Service objects.
```
Service:
{
  serviceID: string,
  img: string, // URL of service image
  title: string, // Title of service
  description: string,
  addOnOnly: boolean
}
```

***

#### coordinatesForAddress_Concierge
Used to get latitude and longitude coordinates for the Address object used in [pullApptSlots](#pullApptSlots).

Parameters:
* _apiKey_ (string)
* _address (Address object):
```
Address:
{
  unit: string, // Ex. "Suite 1"
  street: string, // Ex. "123 Pearl St"
  city: string, // Ex. "Boulder"
  state: string, // Ex. "CO"
  zip: string, // Ex. "12345"
}
```

Returns: Object of latitude and longitude.
```
{
  latitude: number,
  longitude: number
}
```

***

#### pullApptSlots
View available appointment times. This should be used after selecting one or more services.

Parameters:
* _apiKey_ (string)
* _serviceList_ (array of SelectedService objects):
```
SelectedService:
{
  id: string, // ID of selected service
  quantity: number, // Ex. 1
}
```
* _address_ (Address object):
```
Address:
{
  unit: string, // Ex. "Suite 1"
  street: string, // Ex. "123 Pearl St"
  city: string, // Ex. "Boulder"
  state: string, // Ex. "CO"
  zip: string, // Ex. "12345"
  coordinates: { // Get coordinates for an address if needed using coordinatesForAddress_Concierge function
    latitude: number,
    longitude: number
  }
}
```

Returns: Object of appointment slot information by Pro ID.

```
{
  aBc123: { // The key is the Pro's ID
    pro: {
      id: string, // Pro's ID,
      name: string, // Pro's full name,
      firstName: string,
      rating: number, // Pro's average rating from reviews
      numRatings: number,
      img: string, // URL of Pro's profile picture,
      subtotal: number, // The Pro's starting price
      isVIP: boolean, // Indicates whether or not the Pro is a VIP
    },
    apptStartTimes_increments5Min: array of ApptSlot objects, // Available appointment slots in 5 minute intervals
    apptStartTimes_increments15Min: array of ApptSlot objects, // Available appointment slots in 15 minute intervals
    timeslotRanges: array of TimeslotRange objects, // Useful for viewing appointment slots in predictable ranges, like "This Evening", "Tomorrow", etc.
  },
  dEf456: { // Another Pro ID
    ...
  }
}
```
```
ApptSlot:
{
  startTime: date, // Start time of the appointment
  startTime_Formatted: string, // Ex. "5:45 PM"
  startDate_Formatted: string, // Ex. "Tue, Jan 3"
}
```
```
TimeslotRange:
{
  startTime: date, // The starting date of the range
  endTime:date, // The ending date of the range
  startTime_Formatted: string, // Ex. "4:45 PM"
  startDate_Formatted: string, // Ex. "Tue, Jan 3"
  endTime_Formatted: string, // Ex. "8:45 PM"
  endDate_Formatted: string, // Ex. "Tue, Jan 3"
  timezoneOffset: number, // Timezone offset of appointment location in minutes from GMT. Useful for displaying appropriate time and timezone in UI if booking from a different timezone. Ex. 240 for ET.
  increments5Min: array of ApptSlot objects, // Available appointment slots within the time range in 5 minute intervals
  increments15Min: array of ApptSlot objects, // Available appointment slots within the time range in 15 minute intervals
}
```

***

#### lockApptSlot
Locks an appointment slot for consideration before final reservation. This should be used to prevent other viewers from taking an appointment slot while confirming your booking. This must be called before [bookApptSlot](#bookApptSlot).

Parameters:
* _apiKey_ (string)
* _serviceList_ (array of SelectedService objects)
* _address_ (Address object)
* _proID_ (string): The ID of the selected Pro
* _startTime_ (date): The start time of the selected appointment slot

Returns: Object of locked appointment slot information.

```
{
  slotID: string, // ID of the locked appointment slot,
  expiryDate: date, // Must call bookApptSlot before this date is reached, otherwise the appointment slot will be unlocked
  costs: {
    subtotal: number,
    tax: number,
    convenienceFee: number,
    total: number
  }
}
```

***

#### bookApptSlot
Completes an appointment reservation. This must be called after [lockApptSlot](#lockApptSlot).

Parameters:
* _apiKey_ (string)
* _serviceList_ (array of SelectedService objects)
* _address_ (Address object)
* _slotID_ (string): The ID of the locked appointment slot
* _startTime_ (date): The start time of the locked appointment slot
* _clientFullName_ (string): The full name of the client that is receiving services
* _clientEmail_ (string)
* _clientPhone_ (string): Ex. "+11231231234" or "+1-123-123-1234"

Returns: Object of reserved appointment start and end times.

```
{
  apptID: string,
  startTime: date,
  endTime: date
}
```

***

#### cancelApptSlot
Cancels an appointment reservation. This will incur a cancellation fee if within 4 hours of appointment time.

Parameters:
* _apiKey_ (string)
* _apptID_ (string): ID of reserved appointment

Returns: Empty object with status code 200 if cancellation successful.

***

#### pullUpcomingAppts_Concierge
View upcoming scheduled appointments. Provide a client's email to filter for that client's appointments.

Parameters:
* _apiKey_ (string)
* **optional** _clientEmail_ (string): Filter by a client's email

Returns: Object with upcoming appointment info.
```
{
  appts: array of Appointment objects
}
```
```
Appointment:
{
  id: string,
  startTime: date,
  endTime: date,
  address: Address object, // See pullApptSlots documentation for Address definition
  timezone: {
    offset: number, // In minutes from GMT. Ex. 240
    abbr: string, // Timezone abbreviation. Ex. "EDT"
  },
  pro: Pro object,
  status: string, // Status of the appointment. Can be "Unapproved" before the Pro has confirmed the appointment, "Pending" before the appointment takes place, "Completed" after the appointment takes place, "Reviewed" after the client leaves a review, "Cancelled by Client" if the client cancels the appointment,  or "Cancelled by Admin" if cancelled by Shortcut staff.
  parsedCutTypes: string, // Formatted string listing the services in the appointment. Ex. "Barber Cut: 2, Beard Trim: 1"
  serviceTypeList: array of Service objects, // Services provided in the appointment
  conciergeClient: { // Client info
    email: string,
    fullName: string,
    phone: string, // Ex. "(123) 123-1234"
  },
  costs: {
    beforeCouponCost: number, // The subtotal before credits, promos, or tax have been applied.
    creditAmt: number,
    promoAmt: number,
    tax: number,
    charge: number, // The total cost after credits, promos, and tax have been applied.
  },
  customer: { // Payment information
    cardBrand: string,
    last4: string, // Last four digits of the credit card used
    cardholderName: string,
    zip: string,
    id: string, // Payment id
  }
}
```
```
Service:
{
  id: string, // ID of service
  price: number, // Price of one instance of this service
  quantity: number, // Number of instances of this service in the associated appointment
  title: string, // The title of the service. Ex. "Barber Cut"
}
```
```
Pro:
{
  id: string,
  fullName: string,
  firstName: string,
  lastName: string,
  bio: string,
  instagram: string, // The Pro's instagram handle. Ex. "@shortcut"
  cityState: string, // Where the Pro primarily operates. Ex. "Marietta, GA"
  acceptOnlyCustomAddress: boolean, // If the Pro requires clients to go to their selected address
  isoPhoneNumber: string, // Ex. "+11231231234"
  nationalPhoneNumber: string, // Ex. "(123) 123-1234"
  pic: string, // URL of Pro's profile picture
  proType: string, // The type of services that the Pro provides. Ex. "Hair"
  experienceStartYear: number, // The year that the Pro started in their field
  dateLastActiveInMessages: date, // The last time the Pro responded to a message
  averageRating: number, // Pro's average rating from reviews. Ex. 4.95
  numRatings: number,
  curatedReviews: array of Review objects, // Useful for displaying reviews in UI
  vip: boolean, // Indicates whether or not the Pro is a VIP
  startingPrice: number, // The Pro's starting price for their services
  serviceIDs: array of strings, // Lists the IDs of the services the Pro provides. Ex. ["2", "0", "k_0", "w_20"]
  servicePricing: object of service IDs, // The keys of the object are the service IDs that the Pro provides. The values give pricing information for each service.
}

Review:
{
  date: date, // Date of review
  firstName: string, // Name of reviewer. Ex. "John M"
  message: string, // The message the reviewer left
  rating: number, // The rating the reviewer left
  reviewID: string,
  thumbnail: string, // URL of reviewer's profile picture
}
```

***

#### pullPastAppts_Concierge
View past appointments. Provide a client's email to filter for that client's appointments.

Parameters:
* _apiKey_ (string)
* **optional** _clientEmail_ (string): Filter by a client's email

Returns: Object with upcoming appointment info.
```
{
  appts: array of Appointment objects // See pullUpcomingAppts_Concierge documentation for Appointment definition
}
```