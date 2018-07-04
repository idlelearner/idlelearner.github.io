---
layout: post
title:  "Design reviews - good engineering practice"
categories: [Engineering best practice]
tags: [Design, Good Practice, Engineering]
date:   2018-06-25 08:44:21 -0700
---

Design reviews are a nice process to get feedback from other engineers in the company when designing a new system/feature. It is a learning opportunity for the designer and a good collaboration method to design better systems using the expertise of senior folks in the company. I would like to explain the process we follow at WP when developing a new feature or system. These are three components to the design review.  

1.Writing a design document.  
2.Sharing the document and receiving feedback.  
3.Schedule a meeting and go over the design for any flaws/improvements.  

**Writing design document:** 
The engineer who is writing the document should be thoughtful about explaining the product specification and full feature design details so that other engineers who may not be aware of the complete system would still be able to get a full picture and be able to provide feedback on the design.

Below are some basic components that has to be covered in the document.
- Product specification
- High level design
- Alternate designs/considerations
- Assumptions
- Cost of resources.
- Operations - monitoring/logging/deployment.
- Testing
- Milestones/estimates.  

Design document is a good way to document the feature/system at a high level which can used by other engineers to understand to implement/extend the system.

**Sharing and receiving feedback:**
Once the design document is prepared, it is shared to a larger audience who can then comment on various sections of the document. This might help in redesigning/improve the system over the course of review. Engineer who is designing the system should be open to suggestions which will help in improving the system and use the feedback as a good learning opportunity and knowledge sharing.

**Schedule a meeting:**
The meeting has to be scheduled with the smaller group who have suggestions/comments on the design. PMs are also involved in these meeting to give an overview of the business requirement of the feature and help in trade-offs while considering business decisions in system design. The expectation is that whoever is attending the meeting should have gone through the document and have some suggestion. Unresolved comments are discussed and resolved based on healthy discussions. Once all the meeting members agrees on the design, the document is frozen and the development starts.


##### Responsibilities  
###### Designers:
Engineer who is writing the design document should give complete context of the designed system so that reviewers can understand the system well. He/She should use this as a learning opportunity to improve their design skills and use senior engineers knowledge to improve the system designed.

###### Reviewers:
They should be mindful of the changes they are suggesting with the intention of developing well engineered systems and also considering the time of the designer and business implications. Important thing here is to keep the ego aside when discussing system design with other peers. As most of the time, system design considerations are subjective and are respective to systems used vs cost vs development time vs performance, etc, etc.

For more details to consider while designing systems, I would recommend reading [AWS well-architected framework](https://d0.awsstatic.com/whitepapers/architecture/AWS_Well-Architected_Framework.pdf) . This is just an overview of the design review process. This can be changed based on the requirement of the company and engineers available time.




