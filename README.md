Cost Optimization Challenge: Managing Billing Records in Azure Serverless Architecture
We have a serverless architecture in Azure, where one of our services stores billing records in Azure Cosmos DB. The system is read-heavy, but records older than three months are rarely accessed.

Over the past few years, the database size has significantly grown, leading to increased costs. We need an efficient way to reduce costs while maintaining data availability.

Current System Constraints
Record Size: Each billing record can be as large as 300 KB.

Total Records: The database currently holds over 2 million records.

Access Latency: When an old record is requested, it should still be served, with a response time in the order of seconds.

Solution Requirements
Please propose a detailed solution to optimize costs while ensuring the following
Simplicity & Ease of Implementation – The solution should be straightforward to deploy and maintain.

No Data Loss & No Downtime – The transition should be seamless, without losing any records or requiring2 service downtime.

No Changes to API Contracts – The existing read/write APIs for billing records must remain unchanged

Bonus Points
Include an architecture diagram illustrating your proposed solution.

Provide pseudocode, commands, or scripts for implementing core logic (such as data archival, retrieval, and cost optimization st rategies).

Please share the solution in a Github repo. If you are using chatGPT or other LLMs to solve please share the conversation.
Do's and Don'ts
1. Please do not add your answers here as a comment, create a public Github repo and share the link with us over the chat (again not here)
2. It is completely okay if you use chatGPT or other tools but we want to access the conversation to understand the nuances of the solution. The better you interact with the AI tools the better your chances
3. Please go beyond the obvious and think about where the system could break. Consider the different problems we might face, how we would tackle them, and how we would fix them if we were to run your solution in a production environment that could impact thousands of users.
4. It is okay to submit the answer twice within a week
@vikas-t
Author
vikas-t commented on Apr 11 • 
Do's and Don'ts
1. Please do not add your answers here as a comment, create a public Github repo and share the link with us over the chat (again not here)
2. It is completely okay if you use chatGPT or other tools but we want to access the conversation to understand the nuances of the solution. The better you interact with the AI tools the better your chances
3. Please go beyond the obvious and think about where the system could break. Consider the different problems we might face, how we would tackle them, and how we would fix them if we were to run your solution in a production environment that could impact thousands of users.
4. It is okay to submit the answer twice within a week              give answer to paste on git
