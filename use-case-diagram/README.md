# AirBnB Clone - Use Case Diagram

## 📋 Overview
This use case diagram illustrates the interactions between different actors (Guest, Host, Admin) and the AirBnB Clone system. It visualizes all key functionalities and how users interact with the system.

---

## 👥 Actors

### 1. Guest
**Description**: A user who searches for and books properties.

**Capabilities**:
- Register and manage account
- Search and filter properties
- View property details
- Create and manage bookings
- Make payments
- Write reviews and ratings
- Receive notifications

### 2. Host
**Description**: A property owner who lists and manages properties.

**Capabilities**:
- Register and manage account
- Create, edit, and delete property listings
- Upload property photos
- Set property availability
- Manage bookings
- Receive payouts
- Respond to reviews
- Receive notifications

### 3. Admin
**Description**: System administrator with full access.

**Capabilities**:
- Manage all users (guests and hosts)
- Monitor and manage all listings
- Oversee all bookings
- View system analytics
- Monitor payments and transactions
- Generate reports

### 4. Payment Gateway (External System)
**Description**: Third-party payment processor (Stripe/PayPal).

**Interactions**:
- Process guest payments
- Handle host payouts
- Process refunds

### 5. Email Service (External System)
**Description**: Third-party email service (SendGrid/Mailgun).

**Interactions**:
- Send booking confirmations
- Send payment notifications
- Send general email notifications

---

## 🎯 Use Cases by Category

### User Management (6 Use Cases)
| Use Case ID | Use Case Name | Description | Actors |
|-------------|---------------|-------------|--------|
| UC1 | Register Account | Create a new user account as guest or host | Guest, Host |
| UC2 | Login | Authenticate user credentials | Guest, Host, Admin |
| UC3 | Logout | End user session | Guest, Host |
| UC4 | Update Profile | Modify user profile information | Guest, Host |
| UC5 | Reset Password | Request and reset forgotten password | Guest, Host |
| UC6 | OAuth Login | Login using Google/Facebook | Guest, Host |

### Property Management (6 Use Cases)
| Use Case ID | Use Case Name | Description | Actors |
|-------------|---------------|-------------|--------|
| UC7 | Create Listing | Add a new property listing | Host |
| UC8 | Edit Listing | Modify existing property details | Host |
| UC9 | Delete Listing | Remove a property listing | Host |
| UC10 | View Listings | Browse own property listings | Host |
| UC11 | Upload Photos | Add property images | Host |
| UC12 | Set Availability | Manage property booking calendar | Host |

### Search & Booking (8 Use Cases)
| Use Case ID | Use Case Name | Description | Actors |
|-------------|---------------|-------------|--------|
| UC13 | Search Properties | Search for available properties | Guest |
| UC14 | Filter by Location | Filter search results by location | Guest |
| UC15 | Filter by Price | Filter search results by price range | Guest |
| UC16 | Filter by Amenities | Filter by property amenities | Guest |
| UC17 | View Property Details | View detailed property information | Guest, Host |
| UC18 | Create Booking | Book a property for specific dates | Guest |
| UC19 | Cancel Booking | Cancel an existing booking | Guest |
| UC20 | View Booking History | View past and upcoming bookings | Guest, Host |

### Payment System (4 Use Cases)
| Use Case ID | Use Case Name | Description | Actors |
|-------------|---------------|-------------|--------|
| UC21 | Process Payment | Handle guest payment for booking | Guest, Payment Gateway |
| UC22 | Receive Payout | Receive payment from completed booking | Host, Payment Gateway |
| UC23 | Refund Payment | Process booking cancellation refund | Guest, Payment Gateway |
| UC24 | View Transaction History | View all payment transactions | Guest, Host |

### Reviews & Ratings (4 Use Cases)
| Use Case ID | Use Case Name | Description | Actors |
|-------------|---------------|-------------|--------|
| UC25 | Write Review | Submit a property review | Guest |
| UC26 | Rate Property | Give a property rating (1-5 stars) | Guest |
| UC27 | Respond to Review | Reply to guest review | Host |
| UC28 | View Reviews | Read property reviews | Guest, Host |

### Notifications (3 Use Cases)
| Use Case ID | Use Case Name | Description | Actors |
|-------------|---------------|-------------|--------|
| UC29 | Receive Email Notification | Get email notifications | Guest, Host, Email Service |
| UC30 | Receive Booking Confirmation | Get booking confirmation notice | Guest, Host, Email Service |
| UC31 | Receive Payment Notification | Get payment status updates | Guest, Host, Email Service |

### Admin Operations (6 Use Cases)
| Use Case ID | Use Case Name | Description | Actors |
|-------------|---------------|-------------|--------|
| UC32 | Manage Users | Add, edit, delete, suspend users | Admin |
| UC33 | Manage Listings | Monitor and moderate property listings | Admin |
| UC34 | Manage Bookings | Oversee all booking transactions | Admin |
| UC35 | View Analytics | View system statistics and metrics | Admin |
| UC36 | Monitor Payments | Track all financial transactions | Admin |
| UC37 | Generate Reports | Create system reports | Admin |

---

## 🔗 Relationships

### Include Relationships
- **Create Booking** includes **View Property Details**: User must view property details before booking
- **Write Review** includes **Create Booking**: User must have a completed booking to write a review
- **Process Payment** includes **Create Booking**: Payment is part of the booking process

### Extend Relationships
- **OAuth Login** extends **Login**: OAuth is an alternative login method
- **Cancel Booking** extends **Refund Payment**: Cancellation may trigger a refund

### Uses Relationships
- **Process Payment** uses **Payment Gateway**: External system for payment processing
- **Receive Payout** uses **Payment Gateway**: External system for payouts
- **Refund Payment** uses **Payment Gateway**: External system for refunds
- **Email Notifications** use **Email Service**: External system for sending emails

---

## 📊 Actor Interaction Summary

### Guest Journey
1. **Registration & Login**: Register → Login/OAuth Login
2. **Search**: Search Properties → Apply Filters → View Property Details
3. **Booking**: Create Booking → Process Payment → Receive Confirmation
4. **Post-Stay**: Write Review → Rate Property
5. **Management**: View Booking History → Cancel Booking (if needed)

### Host Journey
1. **Registration & Login**: Register → Login
2. **Property Setup**: Create Listing → Upload Photos → Set Availability
3. **Management**: View Listings → Edit Listing → Delete Listing (if needed)
4. **Earnings**: Receive Payout → View Transaction History
5. **Engagement**: Respond to Reviews → View Booking History

### Admin Journey
1. **Login**: Access admin dashboard
2. **Monitoring**: View Analytics → Generate Reports
3. **Management**: Manage Users → Manage Listings → Manage Bookings
4. **Financial**: Monitor Payments → Handle disputes

---

## 🎨 Diagram Components

### Use Case Symbols
- **Oval shapes**: Represent system functionalities (use cases)
- **Stick figures**: Represent actors (users and external systems)
- **Rectangle**: System boundary (AirBnB Clone System)
- **Lines**: Associations between actors and use cases
- **Dashed arrows**: Include, extend, and uses relationships

### Color Coding
- **Light Blue**: Use cases (system functionalities)
- **Light Yellow**: Actors (users)
- **Black lines**: Direct associations
- **Dashed lines**: Special relationships (include, extend, uses)

---

## 📝 Use Case Specifications (Examples)

### UC18: Create Booking

**Primary Actor**: Guest

**Preconditions**:
- Guest is logged in
- Guest has viewed property details
- Property is available for selected dates

**Main Flow**:
1. Guest selects check-in and check-out dates
2. Guest specifies number of guests
3. System validates availability
4. System calculates total price
5. Guest reviews booking details
6. Guest proceeds to payment
7. System processes payment via Payment Gateway
8. System creates booking record
9. System sends confirmation to guest and host
10. Use case ends

**Postconditions**:
- Booking is created in the system
- Payment is processed
- Property dates are blocked
- Notifications are sent

**Alternative Flows**:
- **3a**: Property not available → Display error, suggest alternative dates
- **6a**: Payment fails → Display error, allow retry

---

### UC7: Create Listing

**Primary Actor**: Host

**Preconditions**:
- Host is logged in
- Host has completed profile

**Main Flow**:
1. Host navigates to "Add Listing"
2. Host enters property title and description
3. Host enters location details
4. Host sets pricing
5. Host selects amenities
6. Host uploads property photos
7. Host sets availability calendar
8. System validates all required fields
9. System creates listing
10. System publishes listing
11. Use case ends

**Postconditions**:
- Listing is created and visible to guests
- Host can manage the listing

**Alternative Flows**:
- **8a**: Validation fails → Display errors, allow correction
- **6a**: Photo upload fails → Allow retry or skip

---

## 🔒 Security Considerations

### Authentication
- All use cases (except Registration and Login) require authentication
- JWT tokens used for session management
- OAuth provides alternative secure authentication

### Authorization
- Role-based access control (RBAC) enforces permissions
- Guests cannot access host-only features
- Hosts cannot access admin features
- Admin has elevated privileges

### Data Protection
- Payment information handled by external secure gateway
- Sensitive data encrypted in transit and at rest
- Input validation prevents injection attacks

---

## 📁 Files in This Directory

```
use-case-diagram/
├── README.md                    # This file
├── use-case-diagram.puml        # PlantUML source
└── use-case-diagram.png         # Exported diagram
```

---

## 🛠️ How to Generate the Diagram

### Option 1: Online PlantUML Editor
1. Go to: http://www.plantuml.com/plantuml/uml/
2. Copy the content from `use-case-diagram.puml`
3. Paste into the editor
4. Download as PNG

### Option 2: VS Code Extension
1. Install "PlantUML" extension
2. Open `use-case-diagram.puml`
3. Press `Alt+D` to preview
4. Right-click → Export as PNG

### Option 3: Command Line
```bash
plantuml use-case-diagram.puml
```

---

## 🔄 Implementation Priority

### Phase 1: Core Use Cases (MVP)
1. UC1, UC2: Registration and Login
2. UC7, UC10: Create and View Listings
3. UC13, UC17: Search and View Property Details
4. UC18: Create Booking
5. UC21: Process Payment

### Phase 2: Essential Features
1. UC19, UC23: Cancel Booking and Refunds
2. UC25, UC26: Reviews and Ratings
3. UC29, UC30, UC31: Notifications
4. UC4: Profile Management

### Phase 3: Advanced Features
1. UC32-UC37: Admin Operations
2. UC6: OAuth Login
3. UC11, UC12: Advanced Property Management
4. UC14, UC15, UC16: Advanced Filtering

---

## 📚 Related Documentation

- **Features Documentation**: `../features-and-functionalities/README.md`
- **Database Schema**: `../alx-airbnb-database/`
- **API Documentation**: Coming soon

---

## 👥 Contributors

- **Project**: AirBnB Clone Backend
- **Documentation**: ALX Backend Development Program
- **Diagram Tool**: PlantUML

---

**Last Updated**: October 2024  
**Version**: 1.0