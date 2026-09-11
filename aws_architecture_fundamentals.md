## AWS Cloud Architecture Fundamentals

An Educational Tutorial Lesson on Core AWS Infrastructure

------------------------------

## 📘 Lesson Overview
This tutorial breaks down how computing power, storage, databases, and serverless architectures connect in Amazon Web Services (AWS). 
By the end of this lesson, you will understand how these pieces fit together like a physical computer and how they power full-stack frameworks like Next.js.

------------------------------

## 🗂️ Module 1: The Virtual Computer (EC2 vs. EBS)
To understand the cloud, think of it as breaking a standard laptop apart into separate, modular components connected over a high-speed network.

* Amazon EC2 (Elastic Compute Cloud): This is the brain of your virtual private server. It provides strictly the CPU (processing power) and RAM (short-term memory) [amazon.com]. It executes your application code.
* Amazon EBS (Elastic Block Store): This is the virtual hard drive. It provides persistent block storage where your Operating System (OS), configurations, and long-term data live.

## How They Connect at Boot
When you start a virtual server, a specialized AWS software layer called a hypervisor (AWS Nitro) instantly links your assigned CPU/RAM with your EBS volume via a lightning-fast virtual connection. The CPU copies the OS kernel from the EBS volume directly into RAM, and your server boots up.
## 💡 Core Mechanics to Remember

* CPU & RAM Alignment: The CPU and RAM must physically reside on the exact same physical host machine inside the AWS data center to avoid network latency.
* The OS Lifecycle: Because your OS lives safely on an EBS volume, you can change your EC2 instance type (upgrading or downgrading CPU and RAM capacity) between boots without losing your data or configurations.

------------------------------

## 💾 Module 2: The Two Types of Server Storage
When configuring an EC2 instance, you have access to two radically different storage technologies:

                  ┌── (Wiped on Stop) ──> [ Instance Store ] (Local NVMe on Motherboard)
[ CPU / RAM ] ────┤
                  └── (Saved on Stop) ──> [ Amazon EBS ] (Network-Attached Virtual Disk)

## 1. Amazon EBS (Persistent Storage)

* Location: Resides on a separate storage rack across the data center room, connected via high-speed fiber optics.
* Persistence: Data is permanently saved. If you stop your instance to save money, your files stay safe.

## 2. EC2 Instance Store (Temporary / Ephemeral Storage)

* Location: Physical NVMe SSDs slotted directly into the motherboard of the exact same host machine as your CPU and RAM [amazon.com].
* Performance: Extreme, ultra-low latency speed.
* Persistence: Completely temporary. If you Stop or terminate your instance, the physical disk is immediately wiped clean for security. It only survives standard OS reboots.

------------------------------
## 🗄️ Module 3: Cloud Databases on AWS
AWS offers different ways to manage relational (SQL) databases like PostgreSQL depending on how much maintenance you want to handle.
## 1. Self-Managed Database (EC2 + EBS)
You manually deploy a database engine by installing it on an EC2 instance and saving the tables to an attached EBS volume. You get full root operating system control, but you must manually script backups, security patches, and scaling.
## 2. Managed Relational Database Service (Amazon RDS)
AWS automates all database management. Behind the scenes, standard RDS still launches a compute instance and attaches an EBS volume to hold the database software and user data, but AWS locks down OS access (no SSH) to handle automated backups, patching, and scaling seamlessly on your behalf.
## 3. Decoupled Cloud-Native Storage (Amazon Aurora)
A premium engine within the RDS family. Aurora bypasses EBS entirely. Instead, it separates compute from storage, copying your database records 6 times across 3 data center rooms (Availability Zones) simultaneously onto a massive, auto-expanding shared storage network.

------------------------------

## 🌐 Module 4: Object Storage (Amazon S3)
Unlike EBS, which acts like a raw local hard drive for a single server, Amazon S3 (Simple Storage Service) is an internet-facing Object Storage system.

* How it works: It stores individual files (images, backups, code assets) inside containers called buckets, accessible globally via secure web URLs/APIs.
* Not a Database: S3 does not natively host transactional database engines like PostgreSQL. However, data analytics tools like Amazon Athena can be used to run SQL queries directly across raw CSV or JSON files sitting inside an S3 bucket.

------------------------------

## 🚀 Module 5: Modern Architecture (Next.js, Lambda, and CI/CD)
Modern cloud applications use a combination of these technologies to achieve zero-maintenance, serverless scaling.
## How Next.js Runs on AWS Lambda
When deploying a full-stack Next.js app to AWS (via tools like SST or OpenNext), the framework splits your app automatically:

   1. Static Files (.css, images, HTML): Are stored in Amazon S3 and cached globally on a CloudFront CDN for instant user loading.
   2. Dynamic Files (SSR, API Routes): Execute inside AWS Lambda (which runs code inside lightweight, secure micro-virtual machines called Firecracker).

## The Automated CI/CD Testing Flow
To deploy safely, professional developers use automated services like GitHub Actions rather than maintaining a dedicated VPS to run tests:

[ Git Push to GitHub ] ──> [ Hosted Runner Boots Ubuntu ] ──> [ Runs Docker Test DB ]
                                                                       │
                                                                   (Passes?)
                                                                       │
[ Deploys to Live AWS Stage ] <── [ Builds Production Assets ] <───────┘


   1. Trigger: You push code to GitHub.
   2. Provision: GitHub provisions a temporary Ubuntu Linux cloud runner automatically.
   3. Isolate & Test: The runner spins up a local Docker container containing a temporary test database. Your code runs tests against this clean sandbox.
   4. Build & Ship: If and only if the tests pass, the pipeline compiles your final production assets and calls SST to deploy them to your production AWS account. The temporary environment is then completely destroyed.

------------------------------

## 🎓 Summary Check: Architecture Cheat Sheet

| Service        | Analogy              | Is it Persistent?               | Common Use Case                     |
|----------------|----------------------|---------------------------------|-------------------------------------|
| Amazon EC2     | CPU + RAM            | No (State lost on termination)  | Running applications, microservices |
| Amazon EBS     | Internal Hard Drive  | Yes                             | OS boot drives, standard databases  |
| Instance Store | Built-in Scratch Pad | No (Wiped completely on Stop)   | Caches, temporary log buffers       |
| Amazon S3      | Global Shared Folder | Yes                             | Media assets, user uploads, backups |
| AWS Lambda     | On-Demand Calculator | No (Shuts down after execution) | Running Serverless APIs / SSR pages |

------------------------------


