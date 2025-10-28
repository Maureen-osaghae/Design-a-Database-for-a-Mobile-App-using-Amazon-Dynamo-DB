#  Design and Implementation of a DynamoDB-Based Mobile Application Backend

---

##  1. Introduction

In today’s digital era, social media applications rely heavily on scalable, low-latency databases to manage large volumes of user-generated content.  
This project demonstrates how to design and implement a **mobile social networking application backend** using **Amazon DynamoDB**, a fully managed NoSQL database service by AWS.

The system supports features such as:
- User profiles
- Photo uploads
- Reactions (likes, etc.)
- Following relationships

It uses **DynamoDB’s single-table design** for optimized performance and scalability.

---

##  2. Project Objectives

- Design a scalable, single-table DynamoDB schema for a social networking app.  
- Model entities and relationships efficiently.  
- Implement access patterns for users, photos, reactions, and friendships.  
- Use AWS services including **Cloud9**, **DynamoDB**, and **boto3** (Python SDK).  
- Implement **secondary indexes** and **transactions** for advanced query and data integrity.

---

##  3. Tools and Technologies

| Tool | Description |
|------|--------------|
| **Amazon Web Services (AWS)** | Cloud platform used for deployment |
| **Amazon DynamoDB** | NoSQL database used for data storage |
| **AWS Cloud9 IDE** | Cloud-based integrated development environment |
| **Python 3** | Programming language used for scripting |
| **boto3** | AWS SDK for Python |
| **AWS CLI** | Command line interface for managing AWS services |

---

##  4. System Design

## Entities
- **User**
- **Photo**
- **Reaction**
- **Friendship**

## Relationships
- A **User** can have multiple **Photos**.
- A **Photo** can have multiple **Reactions**.
- A **Friendship** represents a one-way follow relationship between users.

---

##  5. Primary Key Schema

| Entity | Partition Key (PK) | Sort Key (SK) |
|--------|---------------------|---------------|
| **User** | `USER#<USERNAME>` | `#METADATA#<USERNAME>` |
| **Photo** | `USER#<USERNAME>` | `PHOTO#<USERNAME>#<TIMESTAMP>` |
| **Reaction** | `REACTION#<USERNAME>#<TYPE>` | `PHOTO#<USERNAME>#<TIMESTAMP>` |
| **Friendship** | `USER#<USERNAME>` | `#FRIEND#<FRIEND_USERNAME>` |

This composite key design enables efficient data access across all entities in a **single DynamoDB table**.

---

## 6. Implementation Steps

### **Step 1: Setup AWS Cloud9**

1. Go to **AWS Cloud9** in your AWS console.  
2. Create an environment named `DynamoDB Quick Photos`.  
3. Use default settings and launch.

   <img width="955" height="305" alt="image" src="https://github.com/user-attachments/assets/5a38db68-8048-44d0-a50e-200d89b3d187" />

<img width="959" height="422" alt="image" src="https://github.com/user-attachments/assets/383bce03-ebab-401c-b41f-428307c53d82" />

 Choose Next step > Select Create environment 

 <img width="959" height="234" alt="image" src="https://github.com/user-attachments/assets/beee9ef5-5183-43ee-8d5e-9432532c6596" />

 Install Python and dependencies:

      sudo yum install python3 -y
      pip3 install boto3

## 7. Step 2: Download supporting code

     cd ~/environment
      curl -o quick-photos.tar https://s3.amazonaws.com/ddb-labs/quick-photos.tar
      tar -xvf quick-photos.tar

<img width="690" height="395" alt="image" src="https://github.com/user-attachments/assets/969f625f-106d-4453-a86c-5d4824e1e916" />

Run the ls command ls
 
 ● There should be two directories now, application and scripts

 <img width="623" height="73" alt="image" src="https://github.com/user-attachments/assets/146d93bb-4826-43e5-9d77-e0ba4a686fae" />

 Set permissions:

      chmod +x ~/environment/scripts/*.py

## Create the DynamoDB Table

      # scripts/create_table.py
      import boto3
      
      dynamodb = boto3.client('dynamodb')
      
      try:
          dynamodb.create_table(
              TableName='quick-photos',
              AttributeDefinitions=[
                  {"AttributeName": "PK", "AttributeType": "S"},
                  {"AttributeName": "SK", "AttributeType": "S"}
              ],
              KeySchema=[
                  {"AttributeName": "PK", "KeyType": "HASH"},
                  {"AttributeName": "SK", "KeyType": "RANGE"}
              ],
              ProvisionedThroughput={
                  "ReadCapacityUnits": 10,
                  "WriteCapacityUnits": 10
              }
          )
          print("Table created successfully.")
      except Exception as e:
          print("Error creating table:", e)

   Run: 
        
         python3 scripts/create_table.py

## Bulk Load Data

      # scripts/bulk_load_table.py
      import json
      import boto3
      
      dynamodb = boto3.resource('dynamodb')
      table = dynamodb.Table('quick-photos')
      
      items = []
      
      with open('scripts/items.json', 'r') as f:
          for row in f:
              items.append(json.loads(row))
      
      with table.batch_writer() as batch:
          for item in items:
              batch.put_item(Item=item)
Run:
         
         python3 scripts/bulk_load_table.py
         aws dynamodb scan --table-name quick-photos --select "COUNT"


## Access Patterns

| Feature                    | Operation    | Script                                  |
| -------------------------- | ------------ | --------------------------------------- |
| Create / Update / Get User | Read / Write | `fetch_user_and_photos.py`              |
| Upload Photo               | Write        | `bulk_load_table.py`                    |
| View Photos                | Read         | `fetch_user_and_photos.py`              |
| React to Photo             | Write        | `add_reaction.py`                       |
| View Reactions             | Read         | `fetch_photo_and_reactions.py`          |
| Follow User                | Write        | `follow_user.py`                        |
| View Followers             | Read         | `find_following_for_user.py`            |
| View Followed Users        | Read         | `find_and_enrich_following_for_user.py` |


Secondary Index and Transactions

Inverted Index (GSI): Enables reverse lookups for relationships (e.g., users followed by a given user).

Transactions: Used for atomic operations in add_reaction.py and follow_user.py.

## Entity–Relationship Diagram
**Results**

Example Outputs:

      User<jacksonjason -- John Perry>
      Photo<jacksonjason -- 2019-03-30T02:28:42>
      Reaction<kennedyheather -- PHOTO#david25#2019-03-02T09:11:30 -- +1>
      User john42 is now following user tmartinez

he project successfully:

- Created and managed DynamoDB tables and indexes.

- Queried and updated data using boto3.

- Demonstrated transactional and batch operations.

- Implemented efficient single-table design.

## Cleanup

Delete resources after testing:

      python3 scripts/delete_table.py

Delete Cloud9 environment from AWS console to prevent charges.

Conclusion

This project demonstrated the full lifecycle of designing, implementing, and managing a NoSQL data model for a social networking mobile application using Amazon DynamoDB.

---
## Key Takeaways

- Scalable single-table design with composite keys
- Use of secondary indexes for flexible queries
- Transactional writes ensuring consistency
- Practical use of partial normalization
- This design can scale for real-world applications like Instagram-style apps or photo-sharing platforms, providing both flexibility and speed.

---

