# Amazon S3: Day-to-Day Operations with Python

Follow the cells in [S3_Day_to_Day_Operations.ipynb](S3_Day_to_Day_Operations.ipynb) in order. The lab uses sample HR files and a private S3 bucket in `us-east-1`.

## S3 basics

| Term | Example | Meaning |
|---|---|---|
| Bucket | `operations-<account-id>` | The container for objects, created in one AWS Region. |
| Object | The leave policy JSON and its metadata | One item stored in a bucket. |
| Key | `raw/hr/leave-policy.json` | The object's full, case-sensitive name within the bucket. |
| Prefix | `raw/hr/` | The start of a key. S3 displays prefixes as folders, but you don't need to create them. |

## 1. Set up Boto3

Install `boto3` in the notebook environment if needed. Configure AWS credentials before running the lab (for example, with `aws configure`).

`boto3.client("s3", region_name="us-east-1")` creates the Python object used to call S3. It doesn't create a bucket. A separate STS client calls `get_caller_identity()` to find your account ID, which the notebook appends to `operations-` to reduce bucket-name collisions.

`LOCAL = Path("s3_operations")` points to a **local folder** for sample files and downloads. `LOCAL.mkdir(exist_ok=True)` creates it if needed.

## 2. Create the bucket

`s3.create_bucket(Bucket=BUCKET)` creates the lab bucket in `us-east-1`. `s3.list_buckets(Prefix=BUCKET)` finds bucket names starting with the lab name, and the exact-name check prints only your lab bucket.

## 3. Upload files

The notebook first writes `leave-policy.json`, `employees.csv`, and `travel-policy.txt` to the local folder. Then it uploads them to `raw/hr/` using `s3.upload_file(local_path, BUCKET, key)`.

`s3.put_object()` writes `raw/hr/remote-policy.json` directly from a Python dictionary using `json.dumps()`. No local file is needed. The backup loop uploads files directly inside `LOCAL` to `backup/`; those are separate objects with different keys.

## 4. List keys and check metadata

`list_files()` wraps `s3.list_objects_v2()`. `list_files("raw/hr/")` shows only keys that start with `raw/hr/`, while `list_files()` doesn't filter by prefix. Each call returns up to 1,000 objects; use a paginator for larger listings.

`s3.head_object(Bucket=BUCKET, Key="raw/hr/employees.csv")` returns metadata such as `ContentLength` and `LastModified` without downloading the CSV contents.

## 5. Download a file

`s3.download_file(bucket, key, local_path)` saves `raw/hr/employees.csv` as `s3_operations/downloaded-employees.csv`. The following `.read_text()` reads the local copy, not S3.

## 6. Read directly into Python

`s3.get_object(...)["Body"]` gives you a stream you can read without saving a local file:

- **JSON:** `json.loads()` parses the bytes from `raw/hr/leave-policy.json` into a dictionary.
- **CSV:** `io.TextIOWrapper` decodes the stream from `raw/hr/employees.csv` so `csv.DictReader` can turn the rows into dictionaries.
- **Text:** `.read().decode("utf-8")` turns the bytes from `raw/hr/travel-policy.txt` into text.

`get_object()` reads **one exact key**. Reading `raw/hr/leave-policy.json` returns HR-001 only. To read HR-002, make another call for `raw/hr/remote-policy.json`.

## 7. Update the leave policy

S3 doesn't edit individual JSON fields. Read the object, change the Python dictionary, then call `put_object()` with the full JSON and the **same key**. The notebook changes `annual_leave_days` from `38` to `30`, updates `updated_by`, and reads the object back to verify the change. The remote work policy is unchanged.

## 8. Copy and rename

- **Copy:** `copy_object()` reads from `CopySource` (`raw/hr/leave-policy.json`) and writes to the destination `Key` (`processed/hr/leave-policy.json`). Both objects remain.
- **Rename:** For this bucket, copy `raw/hr/travel-policy.txt` to `raw/hr/travel-policy-2026.txt`, then delete the old key only after the copy succeeds.

## 9. Share a temporary link

`generate_presigned_url("get_object", ...)` uses Signature Version 4 to sign a download link for `raw/hr/leave-policy.json` without making the object public. `ExpiresIn` is in **seconds**; the notebook passes `100` (1 minute 40 seconds). Anyone with the full link can use it while it's valid, so share it only with the intended recipient.

## 10. Delete files

Use `delete_object()` to remove the processed copy of the leave policy, or `delete_objects()` to remove several keys at once. To clear `backup/`, list its keys and pass them to `delete_objects()`; deleting under that prefix doesn't touch `raw/hr/`.

## 11. Keep older versions

Versioning is enabled per bucket. Uploading to the same key creates a new current version without discarding the old one. Deleting a key without specifying a version normally adds a **delete marker** that hides the current object without removing its older versions. Stored versions count toward storage costs.

The versioning cell enables versioning on the source bucket. After enabling it for the first time, wait 15 minutes before rerunning the cell. It writes `38` and then `30` annual leave days to `versioning-demo/leave-policy.json` and reads both values back by `VersionId`. The original policy in `raw/hr/` is unchanged.

## 12. Replicate new objects (optional)

S3 can copy new objects asynchronously to a second bucket. Same-Region Replication (SRR) keeps the copy in the same Region; Cross-Region Replication (CRR) puts it in a different Region.

1. Create a second **lab bucket in the same account**, enable versioning on it, and set up an [IAM role that S3 can assume to replicate objects](https://docs.aws.amazon.com/AmazonS3/latest/userguide/setting-repl-config-perm-overview.html). Your notebook identity also needs permission to configure replication and pass that role to S3.
2. In the replication cell, fill in `DEST_BUCKET`, `DEST_REGION`, and `REPLICATION_ROLE_ARN`. The cell checks versioning on both buckets and sets a rule for `replication-demo/`. Run it only on the dedicated source lab bucket: it replaces any existing replication rules there.
3. Once the rule is active, use `s3.put_object(Bucket=BUCKET, Key="replication-demo/check.txt", Body=b"hello")` to upload a test object. Wait for it to appear in the destination bucket before running cleanup.

The rule doesn't backfill objects uploaded before it was set up; use S3 Batch Replication for existing objects. This lab rule also leaves delete-marker replication off, so deleting from the source won't clean up the destination.

## 13. Clean up

Run cleanup **only on the dedicated source lab bucket**. The cell permanently deletes the remaining objects, including versions and delete markers if versioning was used, then calls `delete_bucket()`. If you ran the replication demo, empty and delete the destination lab bucket separately after checking the replica.

## Common errors

| Error | What to check |
|---|---|
| `NoCredentialsError` | Make sure your notebook can find the AWS credentials you configured. |
| `AccessDenied` | Check which identity STS reports and whether it has permission for the specific action. Keep Block Public Access on. |
| `NoSuchKey` | Use `list_files()` to check the full, case-sensitive key, including its prefix. |
| `BucketAlreadyExists` | Choose a different globally unique bucket name. |
| `BucketNotEmpty` | Check for remaining objects, including older versions and delete markers if versioning was enabled. |

**GenAI takeaway:** An ingestion job can read source documents from `raw/hr/` and write derived files to `processed/hr/`. In a real pipeline, preserve the source documents; the overwrite in this lab is just to demonstrate how updates work.
