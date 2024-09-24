# Project9-Demo-Cloudfront

Hosting a Static Website on S3 with CloudFront

### What is AWS CloudFront?

AWS CloudFront is a **Content Delivery Network (CDN)** service. Simply put, CloudFront speeds up the delivery of your content (like images, videos, and files) to users by caching copies of it in various locations around the world. These locations are called **edge locations**.

A CDN helps reduce the latency that occurs when data travels over long distances by bringing content closer to the user. So, when you're using platforms like **YouTube, Instagram, or Amazon**, you're interacting with their CDN every time content loads quickly on your screen.

### CDN in Action: A Simple Example

Let’s say there’s a user in **Australia** who uploads an image or video to **Instagram**. Normally, Instagram might store that file in a **central storage system**. If someone in **India** tries to access the content, without a CDN, it would take a long time because the data has to travel all the way from Australia to the central storage, and then to India.

This results in **latency**, or the delay that happens when a user is waiting for data to load. Latency is affected by the number of **hops** (routers) the request has to go through before it reaches its destination.

For global platforms like Instagram, **latency is not acceptable**. Users expect quick access to their content, regardless of their location.

### Solving Latency: How CloudFront Works

To solve the issue of latency, Instagram, like many platforms, uses a CDN. In AWS, CloudFront acts as the CDN. Here’s how it works:

1. When the person in Australia uploads a video or image to Instagram, it is stored centrally.  
2. CloudFront will then **cache copies** of this content in various **edge locations** around the world.
3. When someone in India tries to access the image, instead of reaching all the way to the central storage in Australia, the request will be routed to the nearest edge location, for example, in **India**. 
4. This reduces the distance the data has to travel, which speeds up the loading time significantly. The same applies to users in the U.S., Europe, or any other region.

### The Edge Locations

CloudFront’s **edge locations** are distributed globally. Instead of every request traveling to the central server, it travels to the nearest edge location. This makes the user experience much faster, ensuring that images, videos, and other content load **almost instantly**.

Let’s visualize it:

- User in Australia uploads an image.
- CloudFront caches that image in edge locations around the world.
- A user in India accesses the image from an edge location in India.
- A user in the U.S. accesses the image from an edge location in North Virginia.
  
This is how CloudFront **accelerates content delivery** by bringing it closer to the users, wherever they may be.

---
### Step-by-Step Detailed Demo: Hosting a Static Website on S3 with CloudFront

### Part 1: Creating an S3 Bucket for Hosting

1. **Log into AWS Console:**
   Navigate to [AWS Management Console](https://aws.amazon.com/console/), and log in with your credentials.

2. **Go to S3 Service:**
   Search for "S3" in the search bar and click on the S3 service.

3. **Create a New Bucket:**
   - Click on the **Create Bucket** button.
   - Provide a unique name for your bucket (e.g., `www.mydomain.com`). Ideally, this should match your domain name.
   - Select a region for your bucket. (Choose the closest region to your users).
   - Leave the rest of the settings as default.
   - Ensure **Block all public access** is checked. We'll configure access through CloudFront later.
   - Enable **Bucket Versioning** (optional). This is useful if you need to keep versions of objects for recovery purposes.
   - Click **Create Bucket**.

4. **Configure Bucket for Static Website Hosting:**
   - Click on the bucket name to open its settings.
   - Go to the **Properties** tab.
   - Scroll down to the **Static Website Hosting** section, click **Edit**, and enable it.
   - Choose **Host a static website**.
   - Specify your **Index Document** (e.g., `index.html`) and **Error Document** (e.g., `error.html`).
   - Click **Save Changes**.

5. **Upload Website Files:**
   - Go to the **Objects** tab in your bucket.
   - Click **Upload** and then **Add files**. Select your static files (HTML, CSS, JS, images, etc.) and upload them.
   - Click **Upload** to complete the process.

---

### Part 2: Setting Up CloudFront (CDN) for Your S3 Bucket

1. **Open CloudFront:**
   - In the AWS Console, search for **CloudFront** in the services search bar and click to open it.

2. **Create a Distribution:**
   - Click on **Create Distribution**.
   - Choose **Web** as the delivery method for your content.
   - In the **Origin Settings**:
     - For **Origin Domain Name**, select your S3 bucket from the dropdown list (e.g., `www.mydomain.com.s3.amazonaws.com`).
     - Ignore the warning about using the website endpoint and continue.
   - **Origin Path** can be left empty unless you're serving content from a subfolder in the bucket.
   - **Origin Access Control**: Select **Create a New OAI (Origin Access Identity)**. This will restrict direct access to your bucket and allow only CloudFront to fetch the content.
   - Check the box that says **Update Bucket Policy**. This allows CloudFront to access your S3 bucket.

3. **CloudFront Settings:**
   - **Viewer Protocol Policy**: Choose whether you want CloudFront to redirect HTTP requests to HTTPS. It’s recommended to select **Redirect HTTP to HTTPS** for security.
   - **Allowed HTTP Methods**: You can leave this at **GET, HEAD** for a static website.
   - **Distribution Settings**:
     - **Cache Settings**: Default cache behavior should work well for a simple website.
     - **Edge Locations**: Select **Use All Edge Locations** if your users are global. If your website is for a regional audience, you can limit it to a specific region (e.g., North America or Europe).
   - **Alternate Domain Name (CNAME)**: If you have a custom domain (e.g., `www.mydomain.com`), add it here.
   - **SSL Certificate**: If you're using a custom domain, choose **Custom SSL Certificate** and use an AWS Certificate Manager (ACM) certificate.
   - **Default Root Object**: Enter `index.html` so that CloudFront knows which file to serve by default.

4. **Create Distribution:**
   - Review all the settings, then click **Create Distribution**.
   - It may take around 10-15 minutes for CloudFront to deploy the distribution.

---

### Part 3: Configure S3 Bucket Policy for CloudFront Access

While CloudFront is deploying, you need to ensure your S3 bucket has the correct permissions.

1. **Set Bucket Policy:**
   - Go back to your S3 bucket.
   - Click on the **Permissions** tab.
   - Scroll down to **Bucket Policy** and click **Edit**.
   - Add the following policy, replacing the placeholders with your bucket name and CloudFront OAI:
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Principal": {
             "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity YOUR_OAI_ID"
           },
           "Action": "s3:GetObject",
           "Resource": "arn:aws:s3:::www.mydomain.com/*"
         }
       ]
     }
     ```
   - This policy allows CloudFront to read objects from your bucket.

2. **Save the Policy** and ensure there are no public access permissions enabled for other users.

---

### Part 4: Testing the Setup

1. **Obtain the CloudFront Domain Name:**
   - Once CloudFront finishes deploying (it can take 10-15 minutes), go back to CloudFront and find your new distribution.
   - Copy the **Domain Name** (e.g., `d123abc.cloudfront.net`).

2. **Access Your Website:**
   - Paste the CloudFront domain name into your browser (e.g., `https://d123abc.cloudfront.net/index.html`).
   - You should see your static website loading via CloudFront.

3. **Custom Domain (Optional)**:
   - If you're using a custom domain (e.g., `www.mydomain.com`), make sure your DNS settings are updated to point to the CloudFront distribution.

---

### Part 5: Clean-Up

If this was just a demo, remember to delete the resources to avoid unnecessary charges:
- **CloudFront Distribution**: Go to the CloudFront console, disable and then delete the distribution.
- **S3 Bucket**: Go to the S3 console, empty the bucket, and then delete the bucket.

---


