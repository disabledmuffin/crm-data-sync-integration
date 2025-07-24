# CRM Data Sync Integration

A real-time data synchronization system that automatically syncs closed sales deals from a pipeline management system to a billing system, demonstrating modern API integration patterns and data transformation techniques.

## Business Problem

When sales teams close deals in their CRM/pipeline system, customer information needs to flow seamlessly to the billing system for invoicing. Manual data entry creates delays, errors, and frustrated customers waiting for invoices.

This integration automates the entire process: when a deal status changes to "closed-won", customer data automatically syncs to the billing system with proper data transformation and duplicate prevention.

## Technical Solution

**Architecture:**
```
Pipeline System (Airtable) → Webhook Trigger → Sync Logic → Billing System (Airtable)
```

**Key Features:**
- ✅ Real-time webhook simulation for deal closure events
- ✅ Intelligent duplicate detection (incremental sync only)
- ✅ Data transformation between different system schemas
- ✅ Error handling and data validation
- ✅ Visual confirmation via Airtable interface

## Technologies Used

- **API Integration:** Postman for orchestration and testing
- **Data Storage:** Airtable (simulating enterprise CRM and billing systems)
- **Authentication:** Bearer token authentication
- **Data Processing:** JavaScript for transformation logic
- **HTTP Methods:** GET, POST, PATCH for CRUD operations

## System Schema

### Pipeline System Fields
- `contact_id` (Number) - Business identifier
- `name` (Text) - Contact name
- `email` (Email) - Contact email
- `phone` (Phone) - Contact phone
- `company` (Text) - Company name
- `deal_value` (Currency) - Deal amount
- `status` (Select) - Deal status (open, qualified, closed-won, closed-lost)

### Billing System Fields
- `customer_id` (Autonumber) - Auto-generated billing ID
- `name` (Text) - Customer name
- `email` (Email) - Customer email
- `phone` (Phone) - Customer phone
- `company` (Text) - Company name
- `billing_address` (Text) - Billing address
- `account_status` (Select) - Account status (active, inactive)
- `pipeline_sync_id` (Number) - Links back to pipeline contact_id
- `contract_value` (Currency) - Contract value

## How It Works

### 1. Data Retrieval
```javascript
// Fetch all pipeline contacts
pm.sendRequest({
    url: pm.environment.get("airtable_pipeline_url"),
    method: 'GET',
    header: {'Authorization': 'Bearer ' + pm.environment.get("airtable_api_key")}
}, function (err, pipelineResponse) {
    // Process response...
});
```

### 2. Duplicate Detection
```javascript
// Find closed-won deals not already in billing
const closedWonContacts = pipelineContacts.filter(contact => 
    contact.fields.status === "closed-won"
);

const existingBillingIds = billingCustomers.map(customer => 
    customer.fields.pipeline_sync_id
);

const newDeals = closedWonContacts.filter(contact => 
    !existingBillingIds.includes(contact.fields.contact_id)
);
```

### 3. Data Transformation
```javascript
// Transform pipeline data for billing system
const billingCustomer = {
    name: contactToSync.fields.name,
    email: contactToSync.fields.email,
    phone: contactToSync.fields.phone,
    company: contactToSync.fields.company,
    billing_address: "123 Main St, Denver, CO", // Default address
    account_status: "active",
    pipeline_sync_id: contactToSync.fields.contact_id,
    contract_value: contactToSync.fields.deal_value // deal_value → contract_value
};
```

## Key Integration Patterns Demonstrated

### Incremental Synchronization
- Only syncs new closed-won deals (not all data every time)
- Prevents duplicate records and reduces API calls
- Maintains data integrity across systems

### Data Field Mapping
- **Field Renaming:** `deal_value` → `contract_value`
- **Field Addition:** Adds `billing_address` and `account_status`
- **Data Linking:** Uses `pipeline_sync_id` to maintain relationships

### Error Handling
- Validates API responses before processing
- Gracefully handles scenarios with no new data
- Logs detailed process information for debugging

### Webhook Simulation
- Demonstrates webhook processing logic
- Shows real-time data transformation capabilities
- Simulates production webhook triggers

## Setup Instructions

### Prerequisites
- Postman (desktop or web version)
- Airtable account (free tier sufficient)

### 1. Airtable Setup
1. Create new Airtable base called "CRM Sync Demo"
2. Create two tables with the schemas shown above:
   - "Pipeline Contacts"
   - "Billing Customers"
3. Get your API key from airtable.com/account
4. Get your base ID from airtable.com/api

### 2. Postman Configuration
1. Import the included Postman collection
2. Create environment with these variables:
   - `airtable_base_id`: Your Airtable base ID
   - `airtable_api_key`: Your Airtable API key
   - `airtable_pipeline_url`: Full pipeline table URL
   - `airtable_billing_url`: Full billing table URL

### 3. Test the Integration
1. Add test contacts to Pipeline Contacts table
2. Set deal status to "closed-won"
3. Run the "Auto Sync - Pipeline to Billing" request
4. Verify customer appears in Billing Customers table

## Demo Flow

1. **Initial State:** Pipeline has deals in various stages
2. **Deal Closure:** Change deal status to "closed-won"
3. **Webhook Trigger:** Run sync request (simulates automatic trigger)
4. **Data Transformation:** Console shows field mapping process
5. **Result Verification:** Customer appears in billing system
6. **Duplicate Prevention:** Run sync again, no duplicate created

## Real-World Applications

This pattern applies to many business integration scenarios:

- **Salesforce → QuickBooks:** Sales to accounting sync
- **HubSpot → NetSuite:** Marketing to ERP integration
- **Shopify → Billing System:** E-commerce to invoice automation
- **Support Tickets → CRM:** Customer service to sales handoff

## Technical Skills Demonstrated

- **API Integration:** RESTful API consumption and orchestration
- **Data Transformation:** Schema mapping between different systems
- **Asynchronous Programming:** Handling multiple API calls and callbacks
- **Business Logic Implementation:** Duplicate detection and conditional processing
- **Error Handling:** Graceful failure management and data validation
- **Authentication:** Secure API access using Bearer tokens

## Files Included

- `CRM_Sync_Collection.postman_collection.json` - Complete Postman collection
- `CRM_Sync_Environment.postman_environment.json` - Environment template
- `README.md` - This documentation
- `demo_screenshots/` - Visual demonstration of the sync process

## Future Enhancements

- **Batch Processing:** Sync multiple deals simultaneously
- **Retry Logic:** Automatic retry for failed sync attempts
- **Bi-directional Sync:** Updates flow both ways between systems
- **Real Webhook Integration:** Connect to actual webhook services
- **Data Validation:** Enhanced field validation and data quality checks
- **Monitoring Dashboard:** Track sync success rates and performance metrics

## License

This project is open source and available under the MIT License.