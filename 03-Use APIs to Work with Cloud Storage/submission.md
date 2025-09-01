# Use APIs to Work with Cloud Storage: Challenge Lab

## Store values in variables

```
export BUCKET_NAME_1=""
export BUCKET_NAME_2=""
export OAUTH2_TOKEN=""
export PROJECT_ID=""
export OBJECT_NAME="demo-image.jpg"
```

## Create bucket settings on json

```
touch bucket1.json && touch bucket2.json && permissions.json

cat > bucket1.json << EOF
{
   "name": "$BUCKET_NAME_1",
   "location": "us",
   "storageClass": "multi_regional"
}
EOF

cat > bucket2.json << EOF
{
   "name": "$BUCKET_NAME_2",
   "location": "us",
   "storageClass": "multi_regional"
}
EOF
```

## Create bucket using curl

- Create Bucket 1

```
curl -X POST --data-binary @bucket1.json \
    -H "Authorization: Bearer $OAUTH2_TOKEN" \
    -H "Content-Type: application/json" \
    "https://www.googleapis.com/storage/v1/b?project=$PROJECT_ID"
```

- Create Bucket 2

```
curl -X POST --data-binary @bucket2.json \
    -H "Authorization: Bearer $OAUTH2_TOKEN" \
    -H "Content-Type: application/json" \
    "https://www.googleapis.com/storage/v1/b?project=$PROJECT_ID"
```

## Upload image

- Upload image to Cloud Shell

In your Cloud Shell session, click on the three-dotted menu icon in the top-right corner. **Click Upload > Choose File**. Select and upload selected $OBJECT_NAME file. This adds the image to your directory.

- Get realpath uploaded file

```
realpath $OBJECT_NAME
```

- Store realpath file to variable

```
export OBJECT=<DEMO_IMAGE_PATH>
```

- Upload image to Bucket 1

```
curl -X POST --data-binary @$OBJECT \
    -H "Authorization: Bearer $OAUTH2_TOKEN" \
    -H "Content-Type: image/png" \
    "https://www.googleapis.com/upload/storage/v1/b/$BUCKET_NAME_1/o?uploadType=media&name=$OBJECT_NAME"
```

## Copy image from bucket 1 to bucket 2

```
curl -X POST -H "Authorization: Bearer $OAUTH2_TOKEN" \
  "https://storage.googleapis.com/storage/v1/b/$BUCKET_NAME_1/o/$OBJECT_NAME/copyTo/b/$BUCKET_NAME_2/o/$OBJECT_NAME"
```

## Make an object publicly accessible

- Update file permissions.json

```
cat > permissions.json << EOF
{
  "entity": "allUsers",
  "role": "READER"
}
EOF
```

- Update object permissions

```
curl -X POST \
  -H "Authorization: Bearer $OAUTH2_TOKEN" \
  -H "Content-Type: application/json" \
  -d @permissions.json \
  "https://storage.googleapis.com/storage/v1/b/$BUCKET_NAME_2/o/$OBJECT_NAME/acl"
```

## Delete the object file and a Cloud Storage bucket (Bucket 1)

- Delete object in Bucket 1

```
curl -X DELETE \
  -H "Authorization: Bearer $OAUTH2_TOKEN" \
  "https://storage.googleapis.com/storage/v1/b/$BUCKET_NAME_1/o/$OBJECT_NAME"
```

- Delete Bucket 1

```
curl -X DELETE \
  -H "Authorization: Bearer $OAUTH2_TOKEN" \
  "https://storage.googleapis.com/storage/v1/b/$BUCKET_NAME_1"
```
