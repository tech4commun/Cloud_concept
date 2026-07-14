# Introduction to Amazon DynamoDB — Hands-On Lab Notes

**Lab:** SPL-71 | **Duration:** ~45 minutes

## Overview
DynamoDB is a fast, flexible, fully managed **NoSQL database** offering consistent, single-digit millisecond latency at any scale. It supports both **document** and **key-value** data models — a great fit for mobile, web, gaming, ad-tech, and IoT applications.

**In this lab you will:**
- Create a DynamoDB table
- Enter data into the table
- Query the table
- Delete the table

---

## Task 1: Create a New Table

Each table requires a **Primary Key (Partition Key)** used to distribute data across DynamoDB servers, and optionally a **Sort Key**. Together, Partition Key + Sort Key uniquely identify each item.

**Steps:**
1. In the AWS Management Console search bar, search for and open **DynamoDB**
2. On the getting started page, choose **Create table**
3. Configure **Table details**:
   - **Table name:** `Music`
   - **Partition key:** `Artist` — type **String**
   - **Sort key (optional):** `Song` — type **String**
4. Leave default settings for indexes and provisioned capacity
5. Choose **Create table**
6. Wait for the table status to change to **Active**

> A "Creating the Music table" message appears while the table provisions.

---

## Task 2: Add Data

**Key concepts:**
- A **table** is a collection of data on a topic
- An **item** = a group of attributes, uniquely identifiable (similar to a row)
- An **attribute** = a fundamental data element (similar to a column) — but unlike relational databases, **each item can have different attributes**
- Only the **Partition Key** (and **Sort Key**, if used) are required — DynamoDB has **no fixed schema** beyond that

**Steps:**
1. Left navigation → **Explore items**
2. Select the **Music** table
3. Choose **Create item**
4. Enter required attributes:
   - **Artist:** `Pink Floyd`
   - **Song:** `Money`
5. Add more attributes via **Add new attribute**:
   - String → **Album** = `The Dark Side of the Moon`
   - Number → **Year** = `1973`
6. Choose **Create item**

> A "The item has been saved successfully" message confirms the save.

**Create a second item** (note the extra `Genre` attribute — demonstrating schema flexibility):

| Attribute Name | Attribute Type | Attribute Value |
|---|---|---|
| Artist | String | John Lennon |
| Song | String | Imagine |
| Album | String | Imagine |
| Year | Number | 1971 |
| Genre | String | Soft rock |

**Create a third item** (note the extra `LengthSeconds` attribute):

| Attribute Name | Attribute Type | Attribute Value |
|---|---|---|
| Artist | String | Psy |
| Song | String | Gangnam Style |
| Album | String | Psy 6 (Six Rules), Part 1 |
| Year | Number | 2011 |
| LengthSeconds | Number | 219 |

> Each item having different attributes without a predefined schema is a core NoSQL flexibility feature. For faster bulk loading, options include AWS Data Pipeline, programmatic loading, or third-party tools.

---

## Task 3: Modify an Existing Item

1. From the item list, select **Psy**
2. Choose **Actions** → **Edit item**
3. Change **Year** from `2011` to `2012`
4. Choose **Save**

> A "The item has been saved successfully" message confirms the update.

---

## Task 4: Query the Table

Two ways to retrieve data: **Query** and **Scan**.

- **Query** — finds items based on Partition Key (and optionally Sort Key); fully indexed, very fast
- **Scan** — looks through every item in the table; less efficient, can be slow on large tables

**Query example:**
1. Left navigation → **Explore items** → select **Music** table
2. Expand **Scan or query items**
3. Select **Query**
4. Enter:
   - **Partition key (Artist):** `Psy`
   - **Sort key (Song):** condition **Equal to**, value `Gangnam Style`
5. Choose **Run**

> The matching song appears quickly — Query is the most efficient way to retrieve data.

**Scan example (with filter):**
1. Select **Scan**
2. Expand **Filters** and configure:
   - **Attribute name:** `Year`
   - **Type:** Number
   - **Condition:** Equal to
   - **Value:** `1971`
3. Choose **Run**

> Only the song released in 1971 (Imagine) is returned.

---

## Task 5: Delete the Table

Deleting the table also deletes all data within it.

1. Left navigation → **Tables**
2. Select the **Music** table
3. Choose **Delete**
4. In the confirmation dialog, type `confirm`
5. Choose **Delete**
6. Use the refresh option to confirm the table is gone

> A "The request to delete the 'Music' table has been submitted successfully" message confirms the deletion request.

---

### Quick Reference Summary
- **Partition Key** (required) + **Sort Key** (optional) = uniquely identify each item
- DynamoDB is **schemaless** beyond the key requirement — items in the same table can have different attributes
- **Query** = fast, indexed, requires Partition Key (+ optional Sort Key condition)
- **Scan** = reads the whole table (or filtered subset), less efficient
- Deleting a table permanently removes all its data