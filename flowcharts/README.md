# Property Booking Process - Flowchart

## Overview
This flowchart visualizes the complete workflow for the **Property Booking Process** in the AirBnB Clone system. It maps out all steps from initial property search to final booking confirmation, including decision points, error handling, and system processes.

## Process Description

### Main Flow
The booking process follows these key stages:

1. **Property Discovery**
   - Guest searches for properties
   - Views property details and availability

2. **Authentication Check**
   - Verifies if guest is logged in
   - Redirects to login/registration if needed

3. **Booking Creation**
   - Guest enters booking details (dates, guests)
   - System calculates total price
   - Displays price breakdown

4. **Payment Processing**
   - Guest selects payment method
   - Payment processed via gateway
   - Transaction recorded

5. **Confirmation**
   - Booking status updated
   - Notifications sent
   - Confirmation generated

## Decision Points

| Decision | Options | Action |
|----------|---------|--------|
| Property Available? | Yes / No | Continue booking or suggest alternatives |
| Guest Satisfied? | Yes / No | Proceed or continue browsing |
| Guest Logged In? | Yes / No | Continue or redirect to login |
| Guest Confirms? | Yes / No | Create booking or abandon |
| Payment Successful? | Yes / No | Confirm booking or retry/cancel |
| Retry Payment? | Yes / No | Retry or cancel booking |

## Process Steps Breakdown

### 1. Pre-Booking Phase
```
- Guest searches for property
- Guest views property details
- Check property availability
- Check guest satisfaction
```

### 2. Authentication Phase
```
- Check if guest is logged in
- If not: redirect to login/register
- Guest completes authentication
```

### 3. Booking Details Phase
```
- Guest enters:
  * Check-in date
  * Check-out date
  * Number of guests
  * Special requests
```

### 4. Price Calculation Phase
```
Calculate:
  - Nightly rate × number of nights
  - Cleaning fee
  - Service fee
  - Taxes
  - Total amount
```

### 5. Payment Phase
```
- Guest selects payment method (Credit Card/Stripe/PayPal)
- Process payment via payment gateway
- Handle payment success/failure
```

### 6. Confirmation Phase
```
On Success:
  - Update booking status to CONFIRMED
  - Update property availability
  - Record payment transaction
  - Send confirmation email to guest
  - Send notification to host
  - Generate booking confirmation
  - Display success message

On Failure:
  - Update booking status to FAILED
  - Log payment error
  - Display error message
  - Offer retry or cancel options
```

## Data Flow

### Inputs
- **Guest Information**: User credentials, profile data
- **Booking Details**: Dates, number of guests, special requests
- **Payment Information**: Payment method, card details
- **Property Data**: Availability, pricing, property ID

### Outputs
- **Booking Record**: Created in database with status
- **Payment Transaction**: Recorded in payment system
- **Email Notifications**: Sent to guest and host
- **Booking Confirmation**: Generated confirmation document
- **Updated Availability**: Property calendar updated

## Error Handling

### Payment Failure
```
1. Log error details
2. Update booking status to FAILED
3. Display user-friendly error message
4. Offer options:
   - Retry with same payment method
   - Try different payment method
   - Cancel booking
```

### Property Unavailable
```
1. Display "Not Available" message
2. Suggest alternative properties
3. Allow guest to modify dates
```

### Authentication Required
```
1. Save booking details temporarily
2. Redirect to login/registration
3. Resume booking after authentication
```

## Database Operations

### Tables Involved
1. **Bookings Table**
   - INSERT: Create new booking record
   - UPDATE: Update booking status

2. **Payments Table**
   - INSERT: Record payment transaction

3. **Properties Table**
   - UPDATE: Update availability

4. **Notifications Table**
   - INSERT: Create notification records

## External Services

### Payment Gateway Integration
- **Stripe API**: Process credit card payments
- **PayPal API**: Process PayPal payments
- Response handling: success/failure

### Email Service Integration
- **SendGrid/Mailgun**: Send transactional emails
- Email types:
  - Booking confirmation (to guest)
  - New booking notification (to host)

## Business Rules

1. **Availability Check**: Property must be available for selected dates
2. **Minimum Stay**: Some properties may require minimum nights
3. **Maximum Guests**: Cannot exceed property capacity
4. **Payment Required**: Booking confirmed only after successful payment
5. **Instant Booking**: No host approval needed (for this flow)

## Security Considerations

- **Authentication**: User must be logged in
- **Payment Security**: PCI-DSS compliant payment processing
- **Data Encryption**: Sensitive data encrypted in transit and at rest
- **Session Management**: Secure session handling
- **CSRF Protection**: Token-based form submissions

## Performance Considerations

- **Availability Check**: Cache property availability for quick lookup
- **Price Calculation**: Optimize for real-time calculation
- **Payment Processing**: Async processing with timeout handling
- **Database Transactions**: Use transactions for data consistency

## Alternative Flows

### Flow 1: Booking with Host Approval
Some properties require host approval before confirmation.

### Flow 2: Split Payment
Allow payment in installments (deposit + remaining).

### Flow 3: Group Booking
Handle bookings for multiple properties simultaneously.

## Testing Scenarios

### Happy Path
```
✓ Available property
✓ Logged-in guest
✓ Valid booking details
✓ Successful payment
✓ Confirmation sent
```

### Error Scenarios
```
✗ Property not available
✗ Guest not logged in
✗ Payment declined
✗ Network timeout
✗ Invalid dates
```

## Success Metrics

- **Conversion Rate**: % of guests who complete booking
- **Payment Success Rate**: % of successful payment transactions
- **Average Booking Time**: Time from search to confirmation
- **Error Rate**: % of bookings with errors

## Future Enhancements

1. **Save for Later**: Allow guests to save incomplete bookings
2. **Price Alerts**: Notify guests of price changes
3. **Flexible Dates**: Suggest nearby dates if unavailable
4. **Multi-Currency**: Support multiple currencies
5. **Discount Codes**: Apply promotional codes

## Related Documentation

- [Features and Functionalities](../features-and-functionalities/)
- [Use Case Diagram](../use-case-diagram/)
- [Data Flow Diagram](../data-flow-diagram/)

## How to Use This Flowchart

1. **For Developers**: Understand the booking implementation flow
2. **For Testers**: Identify test cases and edge cases
3. **For Product Managers**: Visualize user journey
4. **For Designers**: Map UI/UX touchpoints

## Files in This Directory

- `booking-process.puml` - PlantUML source code
- `booking-process.png` - Generated flowchart diagram
- `README.md` - This documentation file

---

**Note**: This flowchart represents the standard instant booking flow. Custom flows may vary based on property settings and host preferences.