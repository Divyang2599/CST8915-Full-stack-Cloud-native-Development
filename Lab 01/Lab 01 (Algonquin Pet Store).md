# CST8915 Lab 1: Algonquin Pet Store on Azure VM

**Student Name:** Divyang Lodariya 
**Student ID:** 041267824

**Course:** CST8915 Full-stack Cloud-native Development  
**Semester:** Winter 2026

## Demo Video

[Watch Demo Video]([text](https://www.youtube.com/watch?v=jZrZi_7XXVU))

---

## Technical Explanations

### Order Service (Node.js)

The Order Service handles all customer orders in the Algonquin Pet Store app. It receives order details from the frontend like what items were bought and quantities, processes them, and queues them using RabbitMQ so other parts of the system can handle tasks like updating inventory or preparing shipment. It's like the checkout system that makes sure no order gets lost, even if the app is busy.

Built with **Node.js**, this service is fast and great at handling many requests at the same time thanks to its asynchronous nature. Node.js works well here because it's easy to connect with RabbitMQ and other tools, and it's perfect for building quick backend APIs. In the microservices setup, this service runs independently on port 3000, receives REST API calls from the Store Front, and sends messages to RabbitMQ for reliable communication with other services.

### Product Service (Rust)

The Product Service manages the entire product catalog — including names, prices, descriptions, images, and current stock levels. It lets the app show available items to customers and updates inventory automatically when orders are placed.

This service is written in **Rust**, a language famous for being super fast, safe, and preventing common bugs like memory issues. Rust is a great choice for this part because it handles data efficiently and reliably, which is important when dealing with inventory that must stay accurate. In the microservices architecture, it runs separately on port 3030 as a REST API. The Store Front calls it to get product lists, and it receives indirect updates to subtract stock after purchases.

### Store Front (Vue.js)

The Store Front is the actual website customers see and use. It's where people browse pets and products, add items to a cart, see prices/totals, and click "Place Order" to buy. It provides a nice, interactive shopping experience in the browser.

Built with **Vue.js**, a lightweight and powerful JavaScript framework, it's perfect for creating dynamic UIs that update instantly. Vue.js is chosen because it's beginner-friendly yet powerful for modern web apps. In the microservices design, this is the frontend layer running on port 8080. It talks to the Product Service to show items and Order Service to submit purchases using simple HTTP requests, while RabbitMQ handles the behind-the-scenes reliable order processing.

---

## Challenges and Learnings (Optional)

[Add your own text here if you want — examples:  
- "The hardest part was opening the correct ports in the Azure NSG and making sure the priorities didn't conflict. I solved it by following the exact priority numbers from the lab instructions."  
- "I learned a lot about microservices — how breaking an app into small services makes it easier to maintain and scale. Also, using RabbitMQ for queuing was eye-opening for handling asynchronous tasks."  
- "Connecting via VS Code Remote-SSH was tricky at first with the SSH key permissions, but chmod 400 fixed it quickly."  

---

Thanks for checking out my submission!
