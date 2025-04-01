# Guiding A Step-by-Step to build ChatBot with Lex-v2 and Amazon Lambda

## Amazon Lex v2 integration with AWS Lambda

![Untitled Diagram](https://github.com/user-attachments/assets/fbc46d3b-03da-4c5e-93cb-6c37ff6d3129)

### Step 1: Log Into Your AWS Account
Log in to your AWS account. In the search bar, type "Lex"

### Step 2: Enter Initial Bot Details

![01](https://github.com/user-attachments/assets/cc7134d5-0af5-4d36-b1b2-ee34ad07706c)

#### 1. In this next step, you must give basic details about the bot. We will create a bot that helps book a hotel - chatbot use case.
#### 2. On IAM permissions - choose
   Create a role with basic Amazon Lex permissions.
#### 4. On Children’s Online Privacy Protection Act (COPPA)- 
we will choose "No" - for this use case.  
Click `NEXT`
#### 5. Enter the language and the audio in which you want the bot to communicate.  
I chose `English(US)` for language and `Mathrew` for the voice
Click `Done`

![02](https://github.com/user-attachments/assets/1a336b6e-3721-4e92-be30-47e940ae559a)

### Step 2: Create Your Intent
1. Define the name of your intent and the description of your intent
   
![03](https://github.com/user-attachments/assets/9fb56543-a347-4a94-8733-555557d92af6)

3. Next comes the `Sample utterance` section, where you can specify the probable questions that the user might ask your bot.
   
![04](https://github.com/user-attachments/assets/1abb69da-43c6-4659-bd6d-0d0d309c1c23)

#### If you double-click on the word itself in the created sentence, you will be able to categorize the type of data that will be filled in the space, like `Nights` as `AMAZON.Number` meaning a number will be placed in that position.

![05](https://github.com/user-attachments/assets/f64baead-6a72-4c2a-a151-2922e1a9e1b8)

4. Next on `Initial response`. Here, I have given “Sure, I can help you book a hotel room ” as shown.

## Lambda Function 
```
import json
from typing import Dict, Any

# Constants
DIALOG_CODE_HOOK = 'DialogCodeHook'
FULFILLMENT_CODE_HOOK = 'FulfillmentCodeHook'

REQUIRED_SLOTS = ['Location', 'CheckInDate', 'Nights', 'RoomType']

def validate(slots: Dict[str, Any]) -> Dict[str, Any]:
    """
    Validate the slots for the hotel booking intent.
    """
    try:
        for slot_name in REQUIRED_SLOTS:
            if not slots.get(slot_name):
                return {
                    'isValid': False,
                    'violatedSlot': slot_name
                }
        return {'isValid': True}
    except Exception as e:
        print(f"Error in validate function: {str(e)}")
        raise

def lambda_handler(event: Dict[str, Any], context: Any) -> Dict[str, Any]:
    """
    Handle the Lambda function invocation for hotel booking.
    """
    try:
        slots = event['sessionState']['intent']['slots']
        intent = event['sessionState']['intent']['name']
        
        print(f"Invocation source: {event['invocationSource']}")
        print(f"Slots: {slots}")
        print(f"Intent: {intent}")

        validation_result = validate(slots)

        if event['invocationSource'] == DIALOG_CODE_HOOK:
            if not validation_result['isValid']:
                return {
                    'sessionState': {
                        "dialogAction": {
                            'slotToElicit': validation_result['violatedSlot'],
                            "type": "ElicitSlot"
                        },
                        "intent": {
                            'name': intent,
                            'slots': slots
                        }
                    }
                }
            
            return {
                "sessionState": {
                    "dialogAction": {
                        "type": "Delegate"
                    },
                    "intent": {
                        'name': intent,
                        'slots': slots
                    }
                }
            }

        if event['invocationSource'] == FULFILLMENT_CODE_HOOK:
            return {
                "sessionState": {
                    "dialogAction": {
                        "type": "Close"
                    },
                    "intent": {
                        'name': intent,
                        'slots': slots,
                        'state': 'Fulfilled'
                    }
                },
                "messages": [
                    {
                        "contentType": "PlainText",
                        "content": "Thanks, I have placed your reservation"
                    }
                ]
            }

        raise ValueError(f"Unknown invocation source: {event['invocationSource']}")

    except Exception as e:
        print(f"Error in lambda_handler: {str(e)}")
        raise

```
