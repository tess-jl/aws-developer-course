# Section 19: AWS Other Services
-overview of services 
-exam will ask very basic level questions about CloudFront, Step Functions and SWF, SES, ACM

### Section Overview
-remember the general ideas of CloudFront, Step Functions and SWF, SES, ACM, no hands on or in depth

### AWS CloudFront
**CloudFront** = a **Content Delivery Network (CDN)**, usually related to S3
* improves read performanced because content is cached at the Edge
* Edge locations = 136 point of presence locations globally--> therefore content delivered to these 136 places globally by this CDN 
* popular with S3 but works with EC2 and LBs
* helps protect against network attacks 
* can have SSL encryption (HTTPS) at edge using ACM
* can use SSL encryption(HTTPS) to talk to my apps --> fully secure, fully encrypted service
* supports RTMP protocol for videos/media


e.g. with bucket on other side of world instead of user talking directly to bucket (will take a long time) user will talk to edge location and then edge location will talk to the bucket, cache the data, and use cache when other users req stuff --> therefore S3 bucket will be cached across the world

### Step Functions and SWF




REVIEW 
-RTMP protocol