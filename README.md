# LCE Integration Documentation

## Overview
Added new LCE (Lowes Commerce Engine) functionality to the SR Chatbot for Item Price and Item-Omniitem data retrieval.

## Features Added

### 1. Item Price API
- **Purpose**: Get pricing information for items
- **Endpoint**: `POST /lce/itemprice`
- **Payload**:
```json
{
  "itemNumber": "5516962",
  "vendorNumber": "28461",
  "modelId": "VG6021STCL6876"
}
```
- **Response**: Base price, final price, retail price

### 2. Item-Omniitem API
- **Purpose**: Get omni item details
- **Endpoint**: `POST /lce/omniitem`
- **Payload**:
```json
{
  "itemNumber": "5516962",
  "storeNumber": "1863"
}
```
- **Response**: Item details, location, model ID, barcode, etc.

## Implementation Changes

### Files Modified/Created

#### 1. Rasa Configuration
- **`data/nlu.yml`**: Added intents with entity extraction
- **`domain.yml`**: Added intents, entities, slots, actions
- **`data/stories.yml`**: Added conversation flows
- **`actions.py`**: Created custom actions (NEW FILE)
- **`requirements.txt`**: Added dependencies (NEW FILE)

#### 2. Java Backend
- **`LceMetricsController.java`**: Added POST endpoints
- **`LceMetricsService.java`**: Added API call methods
- **`RestCallManager.java`**: Added POST request method

#### 3. Testing
- **`test_lce_integration.py`**: API test script (NEW FILE)

## User Interaction Examples

### Basic Usage
```
User: lce
Bot: [Shows LCE options with buttons including Item Price and Item-Omniitem]

User: item price
Bot: [Shows default item pricing data]

User: omniitem
Bot: [Shows default omniitem data]
```

### Advanced Usage with Parameters
```
User: get price for item 1234567 vendor 12345 model ABC123
Bot: [Shows pricing for specified item]

User: omniitem 5516962 1863
Bot: [Shows omniitem data for specified item and store]
```

## Architecture Flow

1. **User Input** → Rasa NLU extracts intent and entities
2. **Entity Extraction** → Parameters stored in slots
3. **Action Trigger** → Custom Python action called
4. **API Call** → Java backend receives POST request
5. **External API** → Java calls Lowes APIs
6. **Response** → Data formatted and sent to user

## Technical Details

### Entity Extraction
- `item_number`: Extracted from user input
- `vendor_number`: Extracted from user input  
- `model_id`: Extracted from user input
- `store_number`: Extracted from user input

### Fallback Values
- Item Price: itemNumber="5516962", vendorNumber="28461", modelId="VG6021STCL6876"
- Omniitem: itemNumber="5516962", storeNumber="1863"

### API Integration
- Uses POST requests with JSON payloads
- Handles errors gracefully
- Returns formatted responses to users

---

# Testing Guide

## Prerequisites
- Java 11+
- Python 3.8+
- Maven
- Rasa 3.6.0

## Step 1: Start Java API Server
```bash
cd SR_Chatbot/ere-ferrari-api
mvn spring-boot:run
```
**Expected**: Server starts on http://localhost:8080

## Step 2: Test API Endpoints Directly

### Test Item Price API
```bash
curl -X POST http://localhost:8080/lce/itemprice \
  -H "Content-Type: application/json" \
  -d '{"itemNumber":"5516962","vendorNumber":"28461","modelId":"VG6021STCL6876"}'
```

### Test Omniitem API
```bash
curl -X POST http://localhost:8080/lce/omniitem \
  -H "Content-Type: application/json" \
  -d '{"itemNumber":"5516962","storeNumber":"1863"}'
```

**Expected**: JSON responses with pricing/item data

## Step 3: Run Python Test Script
```bash
cd SR_Chatbot
python test_lce_integration.py
```
**Expected**: Both API calls succeed with status 200

## Step 4: Setup Rasa Environment
```bash
cd SR_Chatbot/ere-blueberry
pip install -r requirements.txt
```

## Step 5: Start Rasa Action Server
```bash
rasa run actions
```
**Expected**: Actions server runs on http://localhost:5055

## Step 6: Train and Start Rasa Bot
```bash
# In new terminal
cd SR_Chatbot/ere-blueberry
rasa train
rasa shell
```

## Step 7: Test Bot Conversations

### Test 1: Basic LCE Options
```
Input: lce
Expected: Shows buttons for LCE Store, LCE Digital, Order Load Failure, Item Price, Item-Omniitem
```

### Test 2: Default Item Price
```
Input: item price
Expected: Shows pricing data with base price, final price, retail price
```

### Test 3: Default Omniitem
```
Input: omniitem
Expected: Shows item details with item number, location, model ID, barcode
```

### Test 4: Custom Parameters
```
Input: get price for item 1234567 vendor 12345 model ABC123
Expected: Shows pricing for specified parameters
```

### Test 5: Custom Omniitem
```
Input: omniitem 9999999 5555
Expected: Shows omniitem data for specified item and store
```

## Troubleshooting

### Common Issues

#### 1. Java API Not Starting
- Check Java version: `java -version`
- Check Maven: `mvn -version`
- Check port 8080 availability

#### 2. Rasa Actions Failing
- Verify requirements installed: `pip list`
- Check action server logs
- Verify API server is running

#### 3. Entity Extraction Not Working
- Check NLU training data format
- Retrain model: `rasa train`
- Test with exact example phrases

#### 4. API Connection Errors
- Verify localhost:8080 is accessible
- Check firewall settings
- Verify JSON payload format

### Log Locations
- **Java API**: Console output where `mvn spring-boot:run` is executed
- **Rasa Actions**: Console output where `rasa run actions` is executed  
- **Rasa Core**: Console output where `rasa shell` is executed

## Success Criteria
✅ Java API endpoints respond with 200 status
✅ Python test script passes both API tests
✅ Rasa actions server starts without errors
✅ Bot responds to "lce" with option buttons
✅ Bot handles both default and custom parameters
✅ Error messages displayed for failed API calls

## Next Steps
- Add input validation for parameters
- Implement caching for API responses
- Add more detailed error handling
- Create unit tests for actions
- Add logging and monitoring
