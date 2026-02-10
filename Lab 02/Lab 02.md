# CST8915 Lab 2: Refactoring for 12-Factor Compliance

**Student Name:** Divyang Lodariya 
**Student ID:** 041267824

**Course:** CST8915 Full-stack Cloud-native Development  
**Semester:** Winter 2026

## Demo Video

[Watch Demo Video](https://youtu.be/4xBHUWtiOko?si=LA07MC1WRATmLVtZ)

---

## Reflection Questions

1. I took out all the information like database addresses, passwords, API keys, and the web address of the other service that was written directly inside the code and replaced them with environment variable lookups. This follows the Configuration rule wich is never write important settings in the code.For the Backing Services factor, both services now treat databases, message queues, and other dependent service as attached resources accessed solely via configurable credentials stored in the environment.

2. Storing configuration in environment variables keeps code clean, portable, and secure across different environments.When we write passwords, addresses, or keys directly in the code, it’s dangerous if someone might accidentally share them on GitHub. It’s also hard to change anything because we have to edit the code and redeploy every time. By using environment variables keeps our code clean and safe. Environment variables are easy to manage externally and increase saftey.

3. Keeping each microservice in its own Git repository makes them feel like completely separate apps. It clear boundaries and ownership, allowing independent development, testing, release cycles, and deployment schedules for each microservice. If there is a bug or slow change in one service doesn’t block or break the others. In short by keeping everything seprate app become faster, safer, and much easier to grow.

## GitHub Repository

**Order-service** : (https://github.com/Divyang2599/order-service-lab-02)

**Product-service** :(https://github.com/Divyang2599/product-service-lab-02)

**Store-Front**: (https://github.com/Divyang2599/store-front-lab-02)
