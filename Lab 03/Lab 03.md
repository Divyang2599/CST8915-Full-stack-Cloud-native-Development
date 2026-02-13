# CST8915 Lab 3: Deploying the Algonquin Pet Store on Azure

**Student Name:** Divyang Lodariya

**Student ID:** 041267824

**Course:** CST8915 Full-stack Cloud-native Development

**Semester:** Winter 2026

## Demo Video

[Watch Demo Video](https://youtu.be/4xBHUWtiOko?si=LA07MC1WRATmLVtZ)

---

## Reflection Questions

### i. What challenges did you encounter when configuring environment variables in the GitHub Actions workflow?

One of the main challenges was getting the YAML syntax exactly right even small indentation mistakes or misplaced keys under the `env` block caused the GitHub Actions build to fail. Is was missing "(:)" in my .env file and to figure it out it took me almost 2 hours and finally my prfofessor help me in it so I updated my .env file and the Deployed the application and it run Successfully. But it much easier than the lab just need to be careful in very small small commands. I was also missing a comma in my package.json file due to it i also get error during deployement process.

### ii. How does deploying microservices on Azure Web App Service differ from running them locally?

 Deploying to Azure Web App Service involves platform-managed infrastructure in which I configure runtime stacks, environment variables, and deployment sources through the portal instead of managing servers yourself. While in local development I ahve to run and install rust, then python and run every command one by one and I was also getting error in that method as I have not administrator authentication so I have to ru SUDO commands and write password everytime, bt in Azure I feel deployement of app was faster and Azure adds benefits like auto-scaling, built-in monitoring, easier secret management, and no need to maintain VMs which also includes introduces deployment-specific troubleshooting.

### iii. Why is it important to use environment variables for configurations in a cloud environment?

Environment variables are crucial in cloud environments because they keep sensitive data like RabbitMQ connection strings, API keys, or service URLs out of the source code, preventing accidental exposure in public repositories. They also make applications more portable and maintainable, the same codebase can run in development, staging, and production with different configurations just by changing variables, without code changes or redeployment. This aligns with 12-Factor App principles and supports scalability and security in cloud-native setups.

## GitHub Repository

**Order-service** : (https://github.com/Divyang2599/order-service-lab-02)

**Product-service** :(https://github.com/Divyang2599/product-service-L3P)

**Store-Front**: (https://github.com/Divyang2599/store-front-lab-02)
