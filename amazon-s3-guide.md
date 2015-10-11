#Guide to storing images on S3 that can be accessible publicly

###Credentials Setup
We would need the credentials for our bucket to be used by our apps.

1. Go to <b>Security Credentials</b> (Click your name on the top right)

2. Go to the <b>Users Tab</b> and create a user. You can create multipe ones on one go but just create one. The generate access key should be checked.

3. Once youve created a user, click the <b>Show User Security Credentials</b>. <b>You should see Access Key ID and Secret Access Key</b>. Copy that in a piece of paper or something. You will need that later for your app.

###Authorization for the User
A user by default does not have any authorization. You, as the root account, have to grant some to it so it can do something and you can only do that by creating a group and including users to it.

1. Go to <b>Groups Tab</b> and create a new group.

2. Now when you reach the <b>Attach Policy</b> step, look for <b>AmazonS3FullAccess</b> and select that. This will give you full access so you can do anything on S3.

###Creating Buckets

1. On the main page, on <b>Storage and Content Delivery</b>, select <b>S3</b>, then create a bucket.

2. On your bucket, select the <b>Properties Tab</b> and then select <b>Permissions</b>.

3. Now you will see that your username is a grantee. That means that the User you are using has permission on accessing this bucket but what we want is for the public to have a way of accessing the URLs that we will be shipping out from here. Click <b>Add Bucket Policy</b> and then at the bottom of the dropdown, click <b>AWS Policy Generator</b>.

4. On <b>Select Policy Type: S3 Bucket Policy</b>, on <b>Effect: Allow</b> (This means to allow access to resource to specific group). On <b>Principal: * </b>(This means the group that you would be "Effecting" access to is any). On <b>Actions: GetObject</b> (This means that that the group that has access to this bucket can only do GET). On <b>Amazon Resource Name : arn:aws:s3:::name-of-bucket/*</b> (This means to apply the policies we have specified to the specified bucket). Select <b>Add Statement</b> then select <b>Generate Policy</b>. Copy the JSON result and then go back to <b>Properties</b>.The JSON result should sort of look like this:
```
{
  "Id": "Policy1444573048409",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1444572731529",
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::test-dog/*",
      "Principal": "*"
    }
  ]
}
```

5.On <b>Properties</b>, select <b>Permissions</b>, then select <b>Add Bucket Policy</b>. Paste the JSON Result to the Bucket Policy and then select <b>Save</b>

Now images that we upload on this bucket will now be able to be accessed by the public.

###Uploading Files to S3 Bucket.

1. Inside the bucket, select action <b>Action</b> then <b>Upload</b>. Now this will let you select files on your local to be placed inside the bucket. Select any kind of photo and then start the upload.

2. Now when you double click the picture it will give you the URL of the photo which you can use for your website. You now dont have to store images inside your db, let S3 handle it for you.

