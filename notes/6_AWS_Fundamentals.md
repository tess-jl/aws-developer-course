# Section 6: AWS Fundamentals Amazon S3
- S3 powers the biggest websites in the world
-advertized as an infinitely-scaling storage
-used as an integration as well

### S3 Buckets and Objects 
- **Buckets** = giant directories where we store objects (files)
* need to have globally-unique name despite S3 being a regional service 
* naming convention: no uppercase, no underscore, 3-63 char, not an IP, must start with lowercase letter or number 

**Objects** = files 
* identified by a **key** = FULL, absolute path
* no actual concept of directories that objects live in but UI will make it seem that way (it's actually a very very long key name, lots of /)
* **Object Values** = the content of the file, the body --> max size 5TB, if uploading more than 5GB need to use "multi-part upload" 
* **Metadata** = list of text key/value pairs - system or user metadata
* **Tags** = key/value pair, unicode, up to 10, useful for security or lifecyle
* **Version ID** only if versioning is enabled

Create a bucket--> add coffee file to it
-if right click open it will evaluate permission as an AWS user but by clicking the Link (public link) --> 2 ways to open a file
-can create a folder
-have two keys --> coffee.jpg and Image folder with coffee.jpg

### S3 Versioning 
-Versioning = bucket-level setting --> means that if we overwrite a file we will update its version 
-best practice to version buckets(protect against deletes, roll back is possible)
-any file that was not versioned prior to enabling versioning will have the version null

Hands on
-if delete file looks like it's gone but if you show versions see that there is a delete marker on it (not truly deleted)

### S3 Encryption 
