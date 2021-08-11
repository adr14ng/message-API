# Notes

# What is the message object going to look like? What fields does it contain?
Object: Message
Fields: 
    - MessageID (UUID)
    - SenderID (ID)
    - ReceiverID (ID)
    - Message (RichText)
    - Date (Datetime)

# What functions need to be available?
    - GET REQUEST
        - Function 1 
            - Get last 100 messages from recepient
                - Normally in a RDBMS we would be able to write something that would return the TOP 100 SORTBY Datetime
                Where SenderID = X and ReceiverID = Y. For this case I'm using DynamoDB for the sake of ease of use and 
                ease of integration with lambda. 
                - Well, as of 10 minutes from writing the above statement, I've learned of PartiQL, a Query Language 
                supported by DynamoDB.
                - And, 5 minutes after finidng out that it does not support "LIMIT"

                PartiQL Query:

                SELECT *
                FROM messageTable
                WHERE SenderID = X
                AND ReceiverID = Y
                ORDER BY MessageID DESC;

                - After receiving the above data, we can manipulate the data in lambda to take the top 100 messages

                - *SILVER LINING*
                Both get request functions can leverage the query

        - Function 2 
            - Get all messages from recepient for last 30 days
                - After executing the above PartiQL Query, we can find our acceptable date range and filter our as necessary

    - POST REQUEST
        - Post request should only post the message object to the dynamoDB

# NPM Packages used:
Moment.js



# NOTE: !!!
According to PartiQL the statement:
KEY MUST BE WRAPPED IN DOUBLE QUOTES
AND
VALUE MUST BE WRAPPED IN SINGLE QUOTES

(Although the above was only necessary for the query statement, for the post, the statement behaved fine with 
single quotes.)


Example of PartiQL Get:
SELECT * 
FROM messageTable 
WHERE "senderID" = '1' 
AND "receiverID" = '2'

Example of PartiQL Insert:

INSERT INTO messageTable VALUE {
"senderID": '1',
"dateVal":'2021-08-10T09:07:15-07:00',
"messageVal":'Goodbye World',
"receiverID":'2',
"messageID":'1'
}


# Lessons Learned:
    - Although the ease of use for dynamodb in the serverless architecture created for this project made 
    for quick development, there are issues regarding fetching data due to the nosql structure of dynamodb 
    and a more traditional SQL based DB would be better suitted.

# Things to be wary of:
    - Due to the architecture, this may not be the best solution. The amount of requests needing to be sorted via lambda
    could be very high and may not be capable of keeping up with requests if demand was high. This potentially
    can be mitigated by having the function scale
    - Lambda cold starts could affect 
    - There are limitations to API Gateway being able to get and deliver your data as well, so this solution may not work for people who may have a large dataset to work through
    - DynamoDB has "page limits" that may only return a certain amount of items which may not be representative
    of the true set if the set is large


# TODO:
    - The sorting algorithm needs to be fixed, the items appear to be sorting correctly in the function,
    but when returned are returned in the same exact order. 

# Testing:
    - There needs to be tests written for the following functions:
        - In Post:
            - There needs to be a unit test testing, data types allowed to be posted to the database, writing validation
            functions for inputs would also be very helpful
        - In Get:
            - There needs to be a unit test written to check the request call parameter, the functionality only allows for 
            two "last100" and "last30Days". 
            - Each function would need to be tested to be sure it returns the correct dataset
        - Additonal Testing:
            - Load testing should be done on all AWS services to be sure that the requests can be handled and the 
            infrastructure does not fail
                -API Gateway
                -Lambda
    

    