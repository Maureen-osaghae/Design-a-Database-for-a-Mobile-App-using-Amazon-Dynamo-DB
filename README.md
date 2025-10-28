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


The first step of any data modeling exercise is to build a diagram to show the entities in your
application and how they relate to each other.

In our application, we have the following entities:
● User
● Photo
● Reaction
● Friendship

A User can have many Photos, and a Photo can have many Reactions. Finally, the Friendship
entity represents a many-to-many relationship between Users, as a User can follow multiple
Users and be followed by multiple other Users.
With these entities and relationships in mind, our entity-relationship diagram is shown below.

<img width="512" height="296" alt="image" src="https://github.com/user-attachments/assets/f85661bd-8df3-4c56-b4f7-bc8199bb393b" />

Consider user profile access patterns
Now that we have our entity-relationship diagram, consider the access patterns around our
entities. Let’s start with users.

The users of our mobile application will need to create user profiles. These profiles will include
information such as a username, profile picture, location, current status, and interests for a
given user.
Users will be able to browse the profile of other users. A user may want to browse the profile of
another user to see if the user is interesting to follow or simply to read some background on an
existing friend 

Over time, a user will want to update their profile to display a new status or to update their
interests as they change.
Based on this information, we have three access patterns:
- Create user profile (Write)
- Update user profile (Write)
-  Get user profile (Read)

 ## Consider photo access patterns

Now, let’s look at the access patterns around photos.
Our mobile application allows users to upload and share photos with their friends, similar to
Instagram or Snapchat. When users upload a photo, you will need to store information such as
the time the photo was uploaded and the location of the file on your content delivery network
(CDN).
When users aren’t uploading photos, they will want to browse photos of their friends. If they visit
a friend’s profile, they should see the photos for a user with the most recent photos showing
first. If they really like a photo, they can ‘react’ to the photo using one of four predefined
reactions -- a heart, a smiley face, a thumbs up, or a pair of sunglasses. Viewing a photo should
display the current reactions for the photo.
In this section, we have the following access patterns:
- Upload photo for user (Write)
- View recent photos for user (Read)
- React to a photo (Write)
- View photo and reactions (Read)

  ## Friendship access patterns
Finally, let’s consider the access patterns around friendship.
Many popular mobile applications have a social network aspect. 

You can follow friends, view updates on your friends’ activities, and receive recommendations on other friends you may want to follow.
In your application, a friendship is a one-way relationship, like Twitter. One user can choose to
follow another user, and that user may choose to follow the user back. For our application, we
will call the users that follow a user “followers”, and we will call the users that a user is following
the “followed”.
Based on this information, we have the following access patterns:
- Follow user (Write)
- View followers for user (Read)
- View followed for user (Read)

 ## Design the primary key
Let’s consider the different entities, as suggested in the preceding introduction. In the mobile
application, we have the following entities:
● Users
● Photos
● Reactions
● Friendship

These entities show three different kinds of data relationships.
First, each user on your application will have a single user profile represented by a User entity in
your table.
Next, a user will have multiple photos represented in your application, and a photo will have
Multiple reactions. These are both one-to-many relationships.
Finally, the Friendship entity is a representation of a many-to-many relationship. The Friendship
entity represents when one user is following another user in your application. 

It is a many-to-many relationship as one user may follow multiple other users, and a user may have multiple followers.
Having a many-to-many mapping is usually an indication that you will want to satisfy two Query
patterns, and our application is no exception. On the Friendship entity, we have an access
pattern that needs to find all users that follow a particular user as well as an access pattern to
find all of the users that a given user follows.

Because of this, we’ll use a composite primary key with both a HASH and RANGE value. The
composite primary key will give us the Query ability on the HASH key to satisfy one of the query
patterns we need. 

In the DynamoDB API specification, the partition key is called HASH and the
sort key is called RANGE, and in this guide we will use the API terminology interchangeably and
especially when we discuss the code or DynamoDB JSON wire format.

Note that the one-to-one entity -- User -- doesn’t have a natural property for the RANGE value.
Because it’s a one-to-one mapping, the access patterns will be a basic key-value lookup. Since
your table design requires a RANGE property, you can provide a filler value for the RANGE key 

With this in mind, let’s use the following pattern for HASH and RANGE values for each entity type:




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

