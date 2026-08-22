Prepping for the Software Engineer Perception work.

## Job Description

**Who We Are**

Voxel is building the future of Computer Vision and Machine Learning for operations, risk, and safety. We use computer vision and Al to enable existing security cameras to automatically detect hazards and high-risk activities, keep people safe and drive operational efficiencies. Our technology addresses the key cost drivers for workers' compensation, general liability, and property damage, which cost US employers over $500 billion annually.
**What You'll Do**
We are seeking a highly motivated and bright engineer to join our team and devise elegant solutions to challenging problems. This role will focus on designing and implementing systems to support Voxel's ML development. As a part of the perception team, you should be comfortable working in a fast-paced environment and solving problems across the software stack. We require engineers to be versatile and possess the ability to move across projects and tackle new challenges.
**Responsibilities**
• Develop and productionize state-of-the-art computer vision algorithms for object detection, tracking, pose estimation, activity recognition, and anomaly detection in complex indoor environments like warehouses and retail spaces.
• Implement and optimize computer vision techniques for high-throughput, low-latency applications on edge compute devices.
• Streamline ML data ingestion, retraining pipelines, and model redeployment workflows.
Build and maintain reliable, scalable pipelines for continuous training and deployment of ML models in production environments.
• Partner with Sales, Customer Success, and Field Engineering to architect ML-driven solutions tailored to customer needs and deployment contexts.

**Qualifications**
Must-Haves
• Bachelor's degree in Computer Science, Robotics or a related field.
• Software development experience with proficiency in Python, Go-Lang or C++.
• Ability to solve complex and open-ended problems
• Familiarity with version control systems like Git (e.g., branching, merging, pull requests).

Nice-to-Haves
• Master's or PhD in Computer Science, Robotics, or a related field.
• Coursework or research in computer vision or machine learning.
• 1-3 years of experience in software development.
• Experience with Docker, Bazel build system and/or AWS cloud services like EKS, S3, ECS
• Experience designing large, highly available distributed systems with Kubernetes.
• Experience working with monocular cameras - pose estimation, tracking and state estimation, object detection, segmentation.

**Why Join Us?**
Join a visionary team revolutionizing safety and operations, directly impacting the well-being of millions of essential workers. This is your chance to build an extraordinary business and foster a vibrant company culture that demands your absolute best. Alongside Al experts, experienced entrepreneurs, and passionate problem-solvers, you'll play a pivotal role in shaping the company's growth trajectory and market position. Enjoy a competitive salary, benefits, and a dynamic work environment.

## Company Research

Voxel means Volumetric pixel, it's a 3D unit of graphic information. I assume the name comes the fact that you're analyzing 3D environments using not just 2D pixels from the image but also stereo cameras or LiDAR(Light Detection and Ranging) sensors for depth.

Founded in 2020, Voxel is an AI-powered industrial intelligence platform that helps organizations proactively identify and reduce safety hazards, operational risk and inefficiencies. These issues may result from human error or hazardous conditions and can cause financial loss, bodily harm, and supply-chain disruptions.

Raised 44 million as of now and is a team of 90 people and counting.

### EHS Management and Site Intelligence

- EHS(Environmental, Health and Safety) management software is **reactive:** it records incidents and violations, analyzes root causes, and tracks regulatory compliance after events occur. Example: Intelex.
- **Site intelligence is proactive:** Voxel uses existing security cameras, computer vision, and AI for **real-time monitoring**, **risk identification**, and alerts before hazards become accidents.
- The two systems are strongest together: site intelligence exposes underreported **near misses** and supports immediate coaching, while EHS tools provide formal investigation, reporting, and compliance workflows.
- Their combination creates a **comprehensive safety strategy** based on **efficient resource use** and **continuous improvement**, without requiring additional staff.
- Key phrase: prevent injuries and save lives by shifting safety from incident response to preventative action.

### Measuring the ROI of Safety

- Reactive metrics like incident counts, insurance premiums, workers' compensation claims, and lost productivity only show only part of safety's ROI because they cannot measure **losses avoided**.
- The true cost of an injury includes large **indirect, uninsured, and unrecoverable costs**, such as low morale, paperwork, turnover, retraining, and lost productivity; one cited study estimates indirect costs at **2.7× direct medical costs**.
- Traditional audits and generic training are reactive, brief, and difficult to validate. The proposed alternative is **proactive safety intelligence** built on **always-on visibility** and **data-focused safety strategies**.
- Voxel analyzes existing camera footage for ergonomics, environmental hazards, vehicle safety, PPE use, spills, and blocked exits and paths, then aggregates findings into a dashboard and **automatic triage** reports for site-specific coaching.
- Evidence presented: musculoskeletal-injury prevention programs can return **350%**, ergonomic safety can exceed **10:1 ROI**.
- Americold reported a **77% injury reduction**, **$1.1 million in annual savings**, and a **2,000% direct ROI** at one site.
- Key phrase: turn safety from a cost center into a measurable investment by connecting leading indicators and avoided losses to business outcomes.

### Detecting Near Misses

- A **near miss** is an unplanned event that causes no injury, illness, or damage but had the potential to do so; Voxel focuses on interactions between **powered industrial trucks (PITs)**, people, and other PITs.
- When a moving PIT crosses a configurable distance threshold, Voxel creates an incident and notifies the site team **within 60 seconds**, enabling an immediate coaching moment or corrective action.
- Near misses are a critical **leading indicator**: the blog cites **10–100 near misses for every reported injury**, so finding patterns early can prevent more serious incidents.
- Alerts are shareable and reports include timestamps and locations; historical data helps teams identify trends and improve maintenance, operating rules, training, and safety protocols.
- Sites can tune **distance and speed thresholds** for their warehouse environment, making detection specific to local operating conditions.

### Anu's Summary

A Saas company for industries that uses visual content from already installed camera to proactively detect inadequacies(like obstructive object in path) and operational risk(Near Miss incidents like PIT collison). Helps avoid delay, waste and harm in operations.

AGI, Motus, Ceva are Airport Cargo, waherhouses and supply chain management, and global end to end logistic network companies respectively. They are all Voxel clients.

Mission is to take the proactive approach instead of the reactive approach.

Help avoid bodily harm for the workers and save money for the company by reduction in Workers' Compensation(WC) claims.

## Prep Areas and Sources

Step 1: Initial Screen
Step 2: Leetcode Style Coding Assessment.
Step 3: System Design Questions.

## Elevator Pitch

## Followup Questions

1. Is the Site Intelligence system tuned based on which company is using it and which country it's being used in? I imagine safety protocols can differ between companies, sites, and countries. Do you just tune thresholds for one ever evolving software based on the site or Is it common to customize what the system flags? For example, could you configure it so that in a food-processing facility or a plant that regularly uses water-based coolant, a wet floor isn't automatically flagged because it may be part of normal operating conditions?

2. Do you use vision Transformers in the site intelligence systems or would it be too expensive and infeasible because of the extreme token usage of a transformer model analyzing real time video from dozens of cameras?

3. Does the system perform well enough on cameras that don't physical infrastructure for depth perception like LiDAR sensors or stereo cameras.

## Resume Deep Dive
