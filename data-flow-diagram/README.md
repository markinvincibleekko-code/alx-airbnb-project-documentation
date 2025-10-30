# AirBnB Clone - Data Flow Diagram (DFD)

## 📋 Overview
This Data Flow Diagram (DFD) illustrates how data moves through the AirBnB Clone system. It shows the flow of information between external entities, processes, and data stores for all core backend operations.

**DFD Level**: Level 1 (Context-Level Decomposition)

---

## 🎨 DFD Components

### External Entities (Data Sources/Destinations)
| Symbol | Entity | Description |
|--------|--------|-------------|
| 🟨 Rectangle | Guest | Users who search and book properties |
| 🟨 Rectangle | Host | Users who list and manage properties |
| 🟨 Rectangle | Admin | System administrators |
| ⬜ Rectangle | Payment Gateway | External payment processor (Stripe/PayPal) |
| ⬜ Rectangle | Email Service | External email service (SendGrid/Mailgun) |
| ⬜ Rectangle | Cloud Storage | External file storage (AWS S3/Cloudinary) |

### Processes (Data Transformations)
| ID | Process Name | Description |
|----|--------------|-------------|
| 1.0 | User Management | Handles registration, login, authentication, profile updates |
| 2.0 | Property Management | Manages property listings, photos, availability |
| 3.0 | Search & Filter | Processes search queries and filters results |
| 4.0 | Booking Management | Handles booking creation, cancellation, status tracking |
| 5.0 | Payment Processing | Processes payments, refunds, and host payouts |
| 6.0 | Review & Rating | Manages reviews, ratings, and responses |
| 7.0 | Notification System | Sends email and in-app notifications |
| 8.0 | Admin Operations | Handles administrative tasks and reporting |

### Data Stores (Databases)
| ID | Data Store | Contents |
|----|------------|----------|
| D1 | Users Database | User accounts, profiles, credentials, roles |
| D2 | Properties Database | Property listings, details, photos, availability |
| D3 | Bookings Database | Booking records, dates, status, history |
| D4 | Payments Database | Transactions, payment methods, payout records |
| D5 | Reviews Database | Reviews, ratings, comments, responses |

---

## 🔄 Detailed Data Flow Descriptions

### 1.0 User Management Process

#### Input Data Flows:
1. **Guest/Host → P1.0**: Registration Data
   - First name, last name
   - Email address
   - Password
   - Phone number (optional)
   - Role (guest/host)

2. **Guest/Host → P1.0**: Login Credentials
   - Email address
   - Password

3. **Admin → P1.0**: Admin Login
   - Admin credentials
   - Two-factor authentication code

#### Process Operations:
- Validate input data
- Hash passwords (bcrypt)
- Generate JWT tokens
- Verify credentials
- Manage user sessions
- Update profile information

#### Output Data Flows:
1. **P1.0 → D1**: Store User Data
   - User account information
   - Encrypted passwords
   - Profile details

2. **P1.0 → Guest/Host**: Authentication Token
   - JWT access token
   - JWT refresh token
   - User profile data

3. **D1 → P1.0**: User Credentials
   - Stored user data for verification

---

### 2.0 Property Management Process

#### Input Data Flows:
1. **Host → P2.0**: Property Details
   - Title and description
   - Location (address, city, country, coordinates)
   - Price per night
   - Number of bedrooms, bathrooms
   - Property type
   - Amenities list
   - House rules

2. **Host → P2.0**: Property Photos
   - Image files (JPEG/PNG)
   - Photo descriptions

3. **D1 → P2.0**: Host Verification
   - Host identity confirmation
   - Host permissions

#### Process Operations:
- Validate property data
- Upload images to cloud storage
- Generate property listings
- Calculate average ratings
- Manage availability calendar
- Update property information

#### Output Data Flows:
1. **P2.0 → Cloud Storage**: Upload Images
   - Image files
   - Metadata

2. **Cloud Storage → P2.0**: Image URLs
   - Stored image links
   - CDN URLs

3. **P2.0 → D2**: Store Property Data
   - Complete property information
   - Image URLs
   - Availability data

4. **P2.0 → Host**: Property Listings
   - List of host's properties
   - Property statistics
   - Booking information

---

### 3.0 Search & Filter Process

#### Input Data Flows:
1. **Guest → P3.0**: Search Query
   - Location (city, region, country)
   - Check-in date
   - Check-out date
   - Number of guests (adults, children)
   - Price range (min, max)
   - Amenities preferences
   - Property type
   - Sort preferences

#### Process Operations:
- Parse search query
- Apply location filters
- Check date availability
- Filter by price range
- Filter by amenities
- Calculate distance from location
- Sort results (price, rating, distance)
- Paginate results

#### Output Data Flows:
1. **P3.0 → D2**: Query Properties
   - Search parameters
   - Filter criteria

2. **D2 → P3.0**: Available Properties
   - Matching property list
   - Property details

3. **P3.0 → D3**: Check Availability
   - Property IDs
   - Date ranges

4. **D3 → P3.0**: Booking Status
   - Available/unavailable dates
   - Existing bookings

5. **P3.0 → Guest**: Search Results
   - Filtered property list
   - Property previews
   - Pagination metadata

---

### 4.0 Booking Management Process

#### Input Data Flows:
1. **Guest → P4.0**: Booking Request
   - Property ID
   - Check-in date
   - Check-out date
   - Number of guests
   - Special requests
   - Contact information

2. **Host → P4.0**: Booking Approval/Rejection
   - Booking decision
   - Rejection reason (if applicable)

#### Process Operations:
- Verify property exists
- Check date availability
- Validate guest count vs property capacity
- Calculate total price (nights × price + fees)
- Prevent double bookings
- Create booking record
- Update property availability
- Manage booking status
- Handle cancellations

#### Output Data Flows:
1. **P4.0 → D2**: Verify Property
   - Property ID lookup

2. **D2 → P4.0**: Property Details
   - Property information
   - Pricing
   - Capacity

3. **P4.0 → D3**: Check Date Availability
   - Property ID
   - Date range query

4. **D3 → P4.0**: Availability Status
   - Available/booked status
   - Conflicting bookings

5. **P4.0 → D3**: Create Booking
   - Complete booking data
   - Status (pending/confirmed)

6. **P4.0 → P5.0**: Trigger Payment
   - Booking ID
   - Payment amount
   - Guest payment method

7. **P4.0 → P7.0**: Booking Created Event
   - Notification trigger
   - Booking details

8. **P4.0 → Guest/Host**: Booking Details
   - Booking confirmation
   - Booking ID
   - Itinerary

---

### 5.0 Payment Processing Process

#### Input Data Flows:
1. **Guest → P5.0**: Payment Information
   - Payment method (card, PayPal)
   - Billing address
   - Payment amount

2. **P4.0 → P5.0**: Payment Trigger
   - Booking ID
   - Total amount
   - Currency

#### Process Operations:
- Validate payment information
- Calculate fees (service fee, taxes)
- Process payment via gateway
- Handle payment failures
- Store transaction records
- Schedule host payouts
- Process refunds
- Generate receipts

#### Output Data Flows:
1. **P5.0 → Payment Gateway**: Payment Request
   - Payment amount
   - Currency
   - Payment method token
   - Customer information

2. **Payment Gateway → P5.0**: Payment Status
   - Success/failure status
   - Transaction ID
   - Error messages (if failed)

3. **P5.0 → D4**: Store Transaction
   - Transaction details
   - Payment status
   - Timestamps

4. **P5.0 → P4.0**: Payment Confirmation
   - Payment success/failure
   - Transaction ID

5. **P5.0 → Guest**: Payment Receipt
   - Transaction details
   - Receipt PDF
   - Invoice

6. **P5.0 → Payment Gateway**: Payout Request
   - Host payout details
   - Amount after fees
   - Bank account info

7. **Payment Gateway → P5.0**: Payout Confirmation
   - Payout status
   - Transfer ID

8. **P5.0 → Host**: Payout Notification
   - Payout amount
   - Transfer date
   - Bank details

9. **P5.0 → P7.0**: Payment Event
   - Notification trigger
   - Payment details

---

### 6.0 Review & Rating Process

#### Input Data Flows:
1. **Guest → P6.0**: Review Data
   - Overall rating (1-5 stars)
   - Category ratings (cleanliness, accuracy, etc.)
   - Written comment
   - Photos (optional)

2. **Host → P6.0**: Review Response
   - Response text
   - Timestamp

3. **Guest → P6.0**: View Reviews Request
   - Property ID
   - Filters (rating, date)

#### Process Operations:
- Verify booking completion
- Validate review within time window (14 days)
- Store review data
- Calculate average ratings
- Update property rating
- Flag inappropriate content
- Moderate reviews
- Link reviews to bookings

#### Output Data Flows:
1. **P6.0 → D3**: Verify Booking
   - Booking ID
   - Guest ID verification

2. **D3 → P6.0**: Booking Validation
   - Booking status (completed)
   - Guest verification

3. **P6.0 → D5**: Store Review
   - Review data
   - Ratings
   - Comments

4. **P6.0 → P2.0**: Update Property Rating
   - New average rating
   - Review count

5. **P2.0 → D2**: Update Rating
   - Property rating update
   - Review statistics

6. **P6.0 → P7.0**: Review Event
   - Notification trigger
   - Review details

7. **P6.0 → Guest**: Review Confirmation
   - Submission success
   - Review ID

8. **D5 → P6.0**: Reviews List
   - Property reviews
   - Host responses

9. **P6.0 → Guest**: Property Reviews
   - Reviews with ratings
   - Host responses
   - Sorted and filtered

---

### 7.0 Notification System Process

#### Input Data Flows:
1. **P4.0 → P7.0**: Booking Events
   - Booking created
   - Booking confirmed
   - Booking canceled
   - Check-in reminder

2. **P5.0 → P7.0**: Payment Events
   - Payment successful
   - Payment failed
   - Payout received
   - Refund processed

3. **P6.0 → P7.0**: Review Events
   - New review received
   - Review reminder
   - Host response to review

#### Process Operations:
- Queue notifications
- Format email templates
- Personalize messages
- Track delivery status
- Handle failures and retries
- Log notification history
- Manage notification preferences

#### Output Data Flows:
1. **P7.0 → Email Service**: Email Data
   - Recipient email
   - Subject line
   - Email body (HTML/text)
   - Attachments (receipts, etc.)

2. **Email Service → P7.0**: Delivery Status
   - Sent status
   - Delivery confirmation
   - Bounce/failure notices

3. **P7.0 → Guest**: Email Notifications
   - Booking confirmations
   - Payment receipts
   - Review reminders

4. **P7.0 → Host**: Email Notifications
   - Booking requests
   - Payout notifications
   - Review alerts

---

### 8.0 Admin Operations Process

#### Input Data Flows:
1. **Admin → P8.0**: Management Requests
   - User management queries
   - Property moderation actions
   - Booking oversight requests
   - Report generation requests
   - Analytics queries

#### Process Operations:
- Authenticate admin access
- Query all data stores
- Generate reports
- Calculate analytics
- Moderate content
- Manage disputes
- Export data
- Monitor system health

#### Output Data Flows:
1. **P8.0 → D1**: User Management Queries
   - User search
   - Account suspension requests

2. **D1 → P8.0**: User Data
   - User accounts
   - Activity logs

3. **P8.0 → D2**: Property Management Queries
   - Property search
   - Moderation requests

4. **D2 → P8.0**: Property Data
   - Property listings
   - Statistics

5. **P8.0 → D3**: Booking Management Queries
   - Booking search
   - Dispute resolution data

6. **D3 → P8.0**: Booking Data
   - All bookings
   - Status information

7. **P8.0 → D4**: Payment Monitoring Queries
   - Transaction history
   - Revenue reports

8. **D4 → P8.0**: Transaction Data
   - Payment records
   - Financial statistics

9. **P8.0 → D5**: Review Monitoring Queries
   - Review moderation
   - Flagged content

10. **D5 → P8.0**: Review Data
    - All reviews
    - Moderation queue

11. **P8.0 → Admin**: Reports & Analytics
    - User statistics
    - Revenue reports
    - Booking trends
    - System metrics

---

## 📊 Data Dictionary

### User Data
| Field | Type | Description |
|-------|------|-------------|
| user_id | UUID | Unique user identifier |
| email | String | User email address |
| password_hash | String | Encrypted password |
| first_name | String | User's first name |
| last_name | String | User's last name |
| role | Enum | guest, host, admin |
| phone_number | String | Contact number |
| created_at | Timestamp | Account creation date |

### Property Data
| Field | Type | Description |
|-------|------|-------------|
| property_id | UUID | Unique property identifier |
| host_id | UUID | Foreign key to Users |
| title | String | Property title |
| description | Text | Property description |
| location | Object | Address, city, country, coordinates |
| price_per_night | Decimal | Nightly rate |
| amenities | Array | List of amenities |
| images | Array | Image URLs |
| availability | Object | Calendar of available dates |

### Booking Data
| Field | Type | Description |
|-------|------|-------------|
| booking_id | UUID | Unique booking identifier |
| property_id | UUID | Foreign key to Properties |
| guest_id | UUID | Foreign key to Users |
| check_in_date | Date | Check-in date |
| check_out_date | Date | Check-out date |
| total_price | Decimal | Total booking cost |
| status | Enum | pending, confirmed, canceled, completed |
| created_at | Timestamp | Booking creation date |

### Payment Data
| Field | Type | Description |
|-------|------|-------------|
| payment_id | UUID | Unique payment identifier |
| booking_id | UUID | Foreign key to Bookings |
| amount | Decimal | Payment amount |
| currency | String | Currency code (USD, EUR, etc.) |
| payment_method | String | Card, PayPal, etc. |
| transaction_id | String | Gateway transaction ID |
| status | Enum | pending, completed, failed, refunded |
| created_at | Timestamp | Payment date |

### Review Data
| Field | Type | Description |
|-------|------|-------------|
| review_id | UUID | Unique review identifier |
| booking_id | UUID | Foreign key to Bookings |
| property_id | UUID | Foreign key to Properties |
| guest_id | UUID | Foreign key to Users |
| rating | Integer | Overall rating (1-5) |
| comment | Text | Review comment |
| host_response | Text | Host's response (optional) |
| created_at | Timestamp | Review submission date |

---

## 🔐 Data Security Considerations

### Sensitive Data Protection
- **Passwords**: Hashed using bcrypt before storage in D1
- **Payment Information**: Tokenized by Payment Gateway, never stored
- **Personal Data**: Encrypted at rest in databases
- **API Communication**: HTTPS/TLS encryption for all data flows

### Data Access Control
- **Authentication**: JWT tokens validate all process access
- **Authorization**: RBAC ensures users access only permitted data
- **Audit Logging**: All data modifications logged with timestamps

---

## 📈 Data Flow Performance Optimization

### Caching Strategy
- **Search Results**: Cache in Redis (P3.0)
- **Property Data**: Cache frequently accessed properties (D2)
- **User Sessions**: Store in Redis for fast authentication (P1.0)

### Database Optimization
- **Indexing**: Foreign keys, email, property_id, booking dates
- **Query Optimization**: Use prepared statements, avoid N+1 queries
- **Connection Pooling**: Maintain database connection pool

---

## 📁 Files in This Directory

```
data-flow-diagram/
├── README.md                    # This file
├── data-flow.puml              # PlantUML source
└── data-flow.png               # Exported diagram
```

---

## 🛠️ How to Generate the Diagram

### Option 1: Online PlantUML Editor
1. Go to: http://www.plantuml.com/plantuml/uml/
2. Copy content from `data-flow.puml`
3. Paste into editor
4. Download as PNG

### Option 2: Command Line
```bash
plantuml data-flow.puml
```

---

## 🔄 Data Flow Examples

### Example 1: Guest Books a Property

```
1. Guest → P3.0: Search query (location, dates)
2. P3.0 → D2: Query properties
3. D2 → P3.0: Available properties
4. P3.0 → Guest: Search results
5. Guest → P4.0: Booking request
6. P4.0 → D3: Check availability
7. D3 → P4.0: Dates available
8. P4.0 → P5.0: Trigger payment
9. Guest → P5.0: Payment info
10. P5.0 → Payment Gateway: Process payment
11. Payment Gateway → P5.0: Payment success
12. P5.0 → D4: Store transaction
13. P4.0 → D3: Create booking
14. P4.0 → P7.0: Send notification
15. P7.0 → Email Service: Confirmation email
16. Email Service → Guest: Booking confirmation
17. Email Service → Host: New booking alert
```

### Example 2: Host Creates Listing

```
1. Host → P2.0: Property details
2. Host → P2.0: Property photos
3. P2.0 → Cloud Storage: Upload images
4. Cloud Storage → P2.0: Image URLs
5. P2.0 → D1: Verify host
6. D1 → P2.0: Host verified
7. P2.0 → D2: Store property data
8. P2.0 → Host: Listing created confirmation
```

---

## 📚 Related Documentation

- **Features Documentation**: `../features-and-functionalities/README.md`
- **Use Case Diagram**: `../use-case-diagram/README.md`
- **Database Schema**: `../alx-airbnb-database/`

---

**Last Updated**: October 2024  
**Version**: 1.0