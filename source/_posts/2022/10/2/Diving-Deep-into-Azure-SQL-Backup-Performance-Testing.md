---
title: Diving Deep into Azure SQL Backup Performance Testing
date: 2022-10-02 11:32:29
category:
- Cloud Platforms
- Azure
tags:
- Azure SQL
- backup
- performance testing
---

Let's face it: databases are the unsung heroes of the digital world. They quietly store and manage the massive amounts of data that keep our applications running smoothly. But what happens behind the scenes to ensure these databases are performing optimally, especially when it comes to backups?

Recently, we've been doing some serious digging into Azure SQL Backup performance testing, and let me tell you, it's a fascinating world! It's not just about how fast you can back up a database; it's a complex dance of algorithms, configurations, and a bit of "how do we simulate real-world chaos?"

## Why Performance Testing Matters (and why it’s not just about speed)

You might think the main goal of performance testing is to make backups and recoveries as fast as possible. And while speed is definitely a key factor, there's so much more to it. As we've discovered, it's also about:
- **Stability and Reliability**: Ensuring that your database operations are rock-solid, even under heavy load. We're talking about uncovering those pesky flaky issues that only seem to appear at the worst possible time.   
- **Repeatability**: Setting up tests that you can run over and over again, allowing you to compare results and track improvements with confidence.   
- **Insightful Metrics**: Gathering and analyzing data over time to truly understand how your database is behaving. This helps in spotting trends, identifying bottlenecks, and making informed decisions.   
- **Future-Proofing**: Designing tests that can easily incorporate new features and configurations down the line. Because let's be honest, technology never stands still.   

## Two Key Types of Performance Testing
To thoroughly evaluate Azure SQL's capabilities, we focus on two main types of performance testing:

1. **Performance Test on a Single Database:**
    - This type of test dives deep into the performance of backing up and recovering an individual database.
    - A key focus here is ensuring "transactionally consistent backups," which means that the backup captures a consistent snapshot of the database, even if changes are happening during the backup process.
    - We analyze how various factors impact backup performance, such as:
        - Database size   
        - Database pricing tier   
        - The volume of Change Data Capture (CDC) records   
        - The type of data within the database (e.g., alphanumeric, images)   
    - Ultimately, these tests help us understand how different workloads and database characteristics affect backup and recovery operations.
       
2. **Scale Test on Multiple Databases:**
    - While single database tests are crucial, it's also important to see how the system behaves when handling multiple databases simultaneously.
    - The scale test aims to answer the question: "How do our backup and recovery jobs perform when they are run in parallel on multiple databases?"   
    - This introduces additional complexity, as we now need to consider factors like:
        - The number of databases   
        - The types and sizes of the databases   
        - The workloads on each database   
        - The number of CDC records across all databases   
    - These tests help us evaluate the system's ability to handle concurrent operations and identify potential bottlenecks or performance degradation at scale. 

## The Challenge: Simulating Real-World Chaos
One of the biggest challenges in performance testing is recreating the unpredictable nature of real-world use. It's not enough to just back up a pristine database; you need to simulate the constant churn of data, the influx of transactions, and the various demands that users place on the system.

In our testing of Azure SQL Databases and Managed Instances, we focused on transactionally consistent backups, a feature that sets us apart. To make our tests as realistic as possible, we had to get creative, especially when it came to Change Data Capture (CDC) records.   

## Our Weapon: The CDC Record Generator
CDC records are like a log of all the changes made to a database. They're crucial for things like data replication and auditing. But for performance testing, they present a challenge: how do you generate a controlled number of CDC records while a backup is running?

We developed a clever algorithm to tackle this:   
- It gives us precise control over the number of CDC records generated.
- It's designed to be fast, so it doesn't interfere with the backup process.
- It generates a variety of random queries to mimic real-world activity.
- And here's the kicker: it ensures that for every insert, there's a corresponding delete. This keeps the database size consistent, which is essential for accurate testing.
   
This algorithm allows us to simulate different levels of database activity and see how it impacts backup performance.

# Exploring the Tools of the Trade
In our quest for the most effective testing strategy, we explored several tools and libraries. Here's a quick look at what we considered:
- [tpch-dbgen](https://github.com/electrum/tpch-dbgen): This is a tool used to generate data for the TPC-H benchmark, a standard for decision support systems. We found it suitable for generating seed data of a desired size for our databases.   
- [YCSB (Yahoo! Cloud Serving Benchmark)](https://github.com/brianfrankcooper/YCSB): YCSB is another benchmark used for evaluating database systems. While it can generate seed data, it does so based on record count rather than database size. It also supports insert/update workloads for CDC record generation, but not deletes.   
- [Locust.io](https://locust.io): Locust is an open-source load generation tool. However, we found it unsuitable for our needs because it doesn't provide complete control over the queries generated, and it would have introduced significant I/O overhead.
   
Ultimately, we opted for a custom solution for CDC record generation to meet our specific requirements for control, speed, and accuracy. 

# The Results: A More Performant Azure SQL
All this hard work has paid off. Our testing has led to significant performance improvements in Azure SQL Database backups. We're now able to handle large databases and high transaction volumes with greater efficiency.
   
But perhaps the biggest win is that we now have a robust and automated way to measure performance. This allows us to ensure that databases continue to evolve and meet the demands of modern applications.

