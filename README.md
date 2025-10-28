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

## 🎯 2. Project Objectives

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
```bash
sudo yum install python3 -y
pip3 install boto3
