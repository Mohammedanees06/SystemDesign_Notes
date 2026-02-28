🌍 CDN (Content Delivery Network) in System Design

A Content Delivery Network (CDN) is a distributed network of servers located in multiple geographic locations that deliver content to users from the server closest to them.

The goal of a CDN is to reduce latency, improve performance, and increase availability of web applications.

📘 What is a CDN?

A CDN stores cached copies of static content (such as images, videos, CSS, and JavaScript files) across multiple edge servers worldwide.

When a user requests content, the CDN serves it from the nearest server instead of the origin server.

🧠 Simple Definition

A CDN delivers website content from the nearest server to the user to make loading faster and reduce server load.

🔄 How CDN Works
User Request
      ↓
Nearest CDN Server (Edge Server)
      ↓
Serve Cached Content
      ↓
If not cached → Fetch from Origin Server
Step-by-Step Flow

User requests a website resource.
Request goes to the nearest CDN edge server.
If content exists in cache → returned immediately.
If not → CDN fetches from origin server.

CDN stores a opy for future requests.

🧩 Key Components
🔹 Origin Server
Main server where original application and data exist.

🔹 Edge Servers
CDN servers distributed globally that cache content closer to users.

🔹 Cache
Temporary storage used to store frequently requested content.

⚡ Why CDN is Used

Faster content delivery
Reduced latency
Lower load on origin server
Improved user experience
High availability

✅ Advantages of CDN

🔸 Performance Improvement
Content loads faster because it is served from nearby servers.

🔸 Reduced Server Load
Repeated requests are handled by CDN instead of the main server.

🔸 Scalability
Handles large traffic spikes efficiently.

🔸 Reliability & Availability
If one server fails, another edge server can serve content.

🔸 Security
Provides protection against attacks such as DDoS by absorbing traffic.

⚠️ Challenges of CDN

🔸 Cache Invalidation
Updating content across all servers can take time.

🔸 Additional Cost
CDN services introduce operational expenses.

🔸 Dynamic Content Limitations
CDNs are most effective for static assets.

📱 Real-World Example

When opening Instagram:
Images and videos are served from nearby CDN servers.
The main backend server is not contacted for every image request.
This makes feeds load instantly worldwide.

🎯 Key Takeaway

A CDN improves performance by bringing content closer to users through geographically distributed caching servers.

⭐ Simple Memory Line

👉 CDN = Content served from the nearest location.