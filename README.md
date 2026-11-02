# car-service-book
A web app that is part of a course work project in university on MS WEB. It allows the supposed mechanic to look for cars, insert cars, service history, maintenance and what needs to be done, etc.

The mechanic’s point of view

“A customer comes in and says the car is making a noise. I need to pull up the car in seconds and see everything that was ever done to it. Right now it is paper notebooks and phone photos. What I want:”

Find the car fast. By plate number, VIN, or owner name. Typing “CB 12” should narrow the list.
See the full history of one car. Every visit, with date, mileage, what was done, which parts were used, and what it cost, newest first.
Log a new service in under a minute. Pick the car, enter the mileage, add the work items and parts, save. The total must be calculated automatically.
Never forget a reminder. “Oil change was done at 120,000 km, so next one is due at 130,000 km or in 12 months.” I want to see which cars are due soon.
Catch bad data. Mileage that goes down compared to the last visit is almost certainly a typo, and I want a warning. Duplicate plates should be rejected.
Show the customer a clean record. Owners should log in and see only their own cars and history. They cannot edit anything.
Know what money came in. Costs per car and per month, and the ability to export that for the accountant.

Entities

Car: plate number (unique), VIN (unique, 17 characters), make, model, year, fuel type, current mileage, owner.
Owner: this is the Identity user. A user can have many cars. Keep owner contact details (name, phone) on a small profile or on the Car itself, so you do not need a separate entity.
ServiceRecord: the visit. Car, date, mileage at the time, service type, notes, total cost (calculated), status.
ServiceType: oil change, brakes, tyres, annual inspection, diagnostics, etc. Used for filtering and for reminders.
ServiceItem (line item): one row of a visit, either a labour entry or a part. Description, quantity, unit price. The record’s total is the sum of its items.
Part (optional): a small catalogue of parts you use often. I would skip this and treat parts as free-text line items, which is much simpler.

Relationships

One user has many cars (one-to-many).
One car has many service records (one-to-many).
One service record has many items (one-to-many).
Each service record belongs to one service type (one-to-many).
Many-to-many is optional: a service record can have several service types (an oil change plus a brake check in the same visit). It is a good bonus because it is a real many-to-many, and it matches how real visits work. If you want less work, keep one type per record.

Roles

Mechanic (Admin): full CRUD on cars, service records, items and service types. Sees all cars.
Customer (User): registers, claims or is assigned their cars, sees only their own history and upcoming reminders. Read-only.

The simplest way to link customers to cars: the mechanic creates the car and picks the owner from a dropdown of registered users. This avoids a claiming process, which is hard to do securely.

CRUD for E entities

Car, ServiceRecord and ServiceType have full CRUD, so that is 3 entities. ServiceItems are managed from inside the service record page (add and remove rows), which also counts as extra CRUD work.

Pages

Home: for the mechanic, cars due for service soon and recent visits. For a customer, their cars. For guests, a short description and the login button.
About: the project page required by the assignment.
Cars: list with search by plate, VIN, make or owner, filter by fuel type or make, sorting and pagination. Details page showing the full service history and a cost summary.
Service records: create (with a dynamic list of items), edit, delete, details with printable layout.
Service types: simple CRUD.
My Cars: the customer’s view.
Reminders: a page listing cars whose next service is due by mileage or date.
Login, Register, Error, 404.

The logic worth showing in your defence

Total cost is computed from the items (quantity times unit price, summed), never typed by hand. This avoids inconsistent numbers.
Mileage check: a new record cannot have a lower mileage than the car’s latest record. The car’s current mileage updates when a newer record is saved.
Next service due: each service type has an interval in kilometres and months (for example oil change: 10,000 km or 12 months). The reminder is the last record of that type plus the interval. This is the part that makes it feel like a real product, and it is not hard: one query per car.
Access control: a customer who edits the URL to /Cars/Details/7 for someone else’s car must get NotFound or Forbidden. This is the most common security mistake in student projects, so showing it working scores well.
Unique constraints: plate and VIN are unique in the database (a unique index), not just in code.

Validation rules

Plate: required, format check with a regular expression (for Bulgarian plates something like ^[A-Z]{1,2}\d{4}[A-Z]{2}$, but allow some flexibility).
VIN: exactly 17 characters, letters and digits, no I, O or Q.
Year: between 1950 and next year.
Mileage: 0 to 2,000,000, and not lower than the previous record.
Item quantity above 0, unit price 0 or more.
Service date cannot be in the future.

Bonus mapping

One-to-many and many-to-many: covered.
Search, filter, sort, pagination: on Cars and on Service records.
Registration and login: Identity with two roles.
Comments: a mechanic’s notes on each record, which is cheap to add.
Image upload: photos of the car or of a damaged part on a record. Costs more work, so I would leave it for the end.
Export: CSV of a car’s history or of monthly revenue. Very natural for this topic.
Email: reminders by email are the most realistic feature, but also the most expensive. Skip it unless you have time.
JS framework: not needed.

Files and size

It is a little bigger than events and tickets. Expect about 40 files: four to five models, five controllers (Home, Cars, ServiceRecords, ServiceTypes, and a small one for Reminders), around 22 views, 3 or 4 view models, 2 partials (car card, pagination), and the seeder. Estimated effort is 20 to 30 hours, so roughly 5 hours more than events.

Where it is harder than events and tickets

The dynamic list of line items on the service record form (add and remove rows). You can do it with a bit of JavaScript, or with a simpler approach: save the record first, then add items one at a time on its details page. I recommend the simple approach.
Customer data isolation, because every query for a customer must be filtered by their user ID.
The reminder calculation.

Where it is easier

No concurrency problem with seats, like the ticket purchase.
Data is naturally personal, so Identity feels purposeful rather than bolted on.

My recommendation

Build this version: Car, ServiceRecord, ServiceType, ServiceItem, two roles, mileage validation, reminders, search, pagination, CSV export. Skip parts catalogue, image upload and email. It is a strong project, and you can explain every decision from a real mechanic’s workflow.

Tell me if you want this one, and I will give you the same overview as before (exact file list, build order, pitfalls), then we go step by step.
