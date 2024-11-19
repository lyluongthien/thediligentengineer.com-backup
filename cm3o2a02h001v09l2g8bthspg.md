---
title: "Most Asked Questions During My Job Searching Journey"
datePublished: Tue Nov 19 2024 06:16:11 GMT+0000 (Coordinated Universal Time)
cuid: cm3o2a02h001v09l2g8bthspg
slug: most-asked-questions-during-my-job-searching-journey
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/cckf4TsHAuw/upload/7bb4876b4d5bd46486624902e22d5433.jpeg
tags: interview, interview-questions

---

As I progressed through my job search, I encountered some insightful and frequently asked questions that challenged me to reflect on my professional journey. I’m sharing my answers to these questions, which might provide value to others preparing for similar situations.

---

### **What was the proudest feature you have developed?**

One of my proudest achievements was deploying my very first product to production: **the Singapore Loyalty Program application, *sG***. This project was particularly meaningful as it marked a true *0-to-1* journey under challenging circumstances, including being part of a small, under-resourced team.

I took full ownership of the product lifecycle, including:

* **Infrastructure setup**: Configured the backend server and admin dashboard using Django.
    
* **UI implementation**: Built the app’s interface with React Native, delivering a seamless user experience.
    
* **Continuous deployment**: Managed CI/CD pipelines using Jenkins and GitLab CI to ensure regular, stable updates.
    
* **Client collaboration**: Worked closely with stakeholders to gather feedback and improve the application iteratively.
    

The experience was intense—pushing me to the brink of burnout. However, seeing the platform go live and witnessing its impact on businesses and customers was immensely rewarding. This project strengthened my confidence in taking ownership of complex challenges, from architecture to deployment and delivering meaningful outcomes.

---

### **What was the most challenging technical difficulty you have solved?**

One of the most technically challenging problems I faced was building an **ETL pipeline for a data service** while working at GT. The project constraints were formidable:

* Limited resources, including a single PostgreSQL database, one ELK node, and a single EC2 instance.
    
* Over 20 million records for 1,000 users, requiring real-time updates.
    

To overcome these challenges, I:

1. **Designed an optimized database schema** and implemented efficient GraphQL APIs to support fast querying.
    
2. Migrated heavy queries from SQL to **Elasticsearch**, enabling full-text search and large-scale data handling.
    
3. Built a robust synchronization pipeline:
    
    * PostgreSQL → Our PostgreSQL → Elasticsearch, ensuring real-time data updates.
        
4. Set up comprehensive **observability** with Prometheus, Grafana, and ELK to monitor and maintain system reliability.
    

Beyond the ETL pipeline, I also managed:

* A recommender system.
    
* A Salesforce Data Cloud ingestion service.
    
* A payment service integrated with Stripe.
    

This project taught me the art of designing efficient architectures and building scalable, reliable systems, even with constrained resources.

---

### **What was the biggest mistake you made as a developer?**

Reflecting on my journey, one of my biggest mistakes occurred during the early stages of the sG project. As the sole developer initially, I focused on building a simple, easy-to-understand codebase to enable quick onboarding for new team members. However, I overlooked the importance of establishing **linting rules and coding conventions** from the outset.

When the team expanded, this oversight caused significant friction:

* Code conflicts became frequent due to differing IDE settings and formatting styles.
    
* This slowed down development and impacted team efficiency.
    

To address this, I implemented:

* Standardized **linting rules**.
    
* A comprehensive **coding style guide**.
    
* **Pre-commit hooks** to enforce consistency automatically.
    

Since then, I’ve prioritized setting up foundational tools and processes early in every project to ensure smoother collaboration and maintainability as teams scale.

### **General Tips for Answering These Questions**

1. **Use the STAR method**: This structured approach helps you organize your thoughts and deliver concise, impactful answers.
    
2. **Be specific**: Avoid vague statements—provide details about tools, technologies, and strategies you used.
    
3. **Quantify outcomes**: Back up your answers with measurable results (e.g., "improved system performance by 30%").
    
4. **Highlight transferable skills**: Even if the experience is from a different domain, emphasize skills like problem-solving, teamwork, or ownership.
    

---

Reflecting on these experiences has been a valuable exercise for me during this job search. Each of these challenges helped me grow as an engineer, and I hope sharing them here provides insights or inspiration to others navigating similar paths.

If you have thoughts or similar experiences, I’d love to hear them!

**Cheers,**  
**Kai**