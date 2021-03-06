# Dining Concierge AI Agent
-------------------------------------------------------------------------------------------------------------------------------------------------------------------

S3 Bucket URL : http://jk4534ur2136.s3-website-us-east-1.amazonaws.com/ (Currently Pulled down due to AWS limits)

--------------------------------------------------------------------------------------------------------------------------------------------------------------------
ARCHITECTURE DIAGRAM
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
![Screenshot 2021-10-12 at 12 46 06 AM](https://user-images.githubusercontent.com/91032192/136893038-af12430b-39f5-42ab-a8a4-7754c38163ab.png)

Note : We use SES in place of SNS.

--------------------------------------------------------------------------------------------------------------------------------------------------------------------
COMPONENTS
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
Lambdas : 

LF0 - This uses the request-response model defined in the Swagger API Specification to perform the chat operation with the lex bot.

LF1 - This is used by the lex bot as an initialization and validation code hook for 'DiningSuggestionsIntent' and a fulfillment code hook for 'GreetingIntent', 'DininingSuggestionsIntent' and 'ThankYouIntent'. This lambda also adds the data gathered from the user into an SQS queue, Q1.

Checks Performed For Slots : 

1. Location - The Amazon.US_CITY data type used performs a lot of validation on its own and in that case, our customized validation is not invoked, but in case it does, we only accept location input to be 'manhattan'.

2. Cuisine - Currently, we support cuisines : indian, mexican, italian, spanish, chinese and japanese.

3. Number of People - Number of people should be more than 0 and cannot be negative.

4. Date of Reservation - We don't let people make reservations before the present date (like yesterday etc.). Some predefined validations are performed on user input because the user input is of type AMAZON.Date and in this case our custom validator is not invoked.

5. Time of Reservation - We don't let people make reservations before the current time. Some predefined validations are performed on user input because the user input is of type AMAZON.Time and in this case our custom validator is not invoked.

6. Email Address - We are performing a regex check in our custom validator, however some predefined validations are performed on user input because the user input is of type AMAZON.EmailAddress and in this case our custom validator is not invoked.

LF2 - This is used to fetch data from the SQS queue Q1, gets 3 random restaurant recommendations for the cuisine collected through conversation from ElasticSearch and DynamoDB and sends them over an email, using SES.

--------------------------------------------------------------------------------------------------------------------------------------------------------------------
CloudWatchTrigger : 

[name-of-trigger] - It runs every minute and invokes LF2 as a result.

--------------------------------------------------------------------------------------------------------------------------------------------------------------------
SQS QUEUE : 

Q1 - Based on the parameters collected from the user, we push information collected from user (location, cuisine, noOfPeople, dateofreservation, timeofreservation, emailaddress) to this queue.

The parameters are then polled out from this queue for querying our dynamodb table, elastic search and composing the email to be sent to the user.

--------------------------------------------------------------------------------------------------------------------------------------------------------------------
SES Service :

The email with three restaurant recommendations with their addresses will be sent to the user email id, SUBJECT to email address verification due to sandbox limitations.

--------------------------------------------------------------------------------------------------------------------------------------------------------------------
Lex : 

DiningConcierge - It has 3 intents :
1. GreetingIntent
2. DiningSuggestionsIntent
3. ThankYouIntent

Based on user input from U/I the handler for each intent is invoked.


1. GreetingIntent :

Utterances - hi, hello, hey

Slots - None

Initialization and Validation Code Hook - None

Fulfillment - LF1

Response from LF1 - Hi there, how can I help?

2. DiningSuggestionsIntent :

Utterances - food, restaurant, restaurants

Slots - location (AMAZON.US_CITY), cuisine (AMAZON.AlphaNumeric), noOfPeople (AMAZON.NUMBER), dateofReservation (AMAZON.DATE), timeofReservation (AMAZON.TIME), emailaddress (AMAZON.EmailAddress)

Initialization and Validation Code Hook - LF1

Fulfillment - LF1

Response from LF1 - Based on slot input.

3. ThankYouIntent :

Utterances - thank you, thanks, thank

Slots - None

Initialization and Validation Code Hook - None

Fulfillment - LF1

Response from LF1 - You're welcome.


