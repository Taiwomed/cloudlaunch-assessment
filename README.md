# CloudLaunch Assessment

## Task 1: S3 Static Website and IAM

### S3 Buckets
- **cloud-launchsite-bucket**: Public, static website served via CloudFront.
  - S3 URL: `http://cloud-launchsite-bucket.s3-website.eu-north-1.amazonaws.com` (Access Denied due to OAC)
  - CloudFront URL: `d6bjvsbuq9pv.cloudfront.net/index.html`
- **cloud-launch-private-bucket**: Private, read/write for `cloudlaunch-user`.
- **cloudlaunch-visible-only-bucket9**: Private, list-only for `cloudlaunch-user`.

### IAM User
- **User**: `cloudlaunch-user`
- **Access**: Programmatic and console access.
- **Console Login URL**: `https://204049628066.signin.aws.amazon.com/console`
- **Policies**: 
  - `CloudLaunchS3Policy`
  - `IAMUserChangePassword`
  - `CloudLaunchVPCReadOnlyPolicy`
  - CloudLaunchS3Policy:
`{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListAllBuckets",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:ListAllMyBuckets"
            ],
            "Resource": [
                "arn:aws:s3:::cloud-launchsite-bucket",
                "arn:aws:s3:::cloud-launch-private-bucket",
                "arn:aws:s3:::cloudlaunch-visible-only-bucket9",
                "arn:aws:s3:::*"
            ]
        },
        {
            "Sid": "SiteBucketRead",
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::cloud-launchsite-bucket/*"
        },
        {
            "Sid": "PrivateBucketReadWrite",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::cloud-launch-private-bucket/*"
        }
    ]
}`
- IAMUserChangePassword:
  `{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:ChangePassword",
                "iam:GetAccountPasswordPolicy"
            ],
            "Resource": "*"
        }
    ]`
- CloudLaunchVPCReadOnlyPolicy:
  `{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VPCReadOnly",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeVpcs",
                "ec2:DescribeSubnets",
                "ec2:DescribeRouteTables",
                "ec2:DescribeInternetGateways",
                "ec2:DescribeSecurityGroups"
            ],
            "Resource": "*"
        }
    ]
}`

### Permission Tests
- **List**: All buckets listed in S3 console as `cloudlaunch-user`.
- **Read**: `index.html` downloaded from `cloud-launchsite-bucket`.
- **Read/Write**: Files uploaded/downloaded in `cloud-launch-private-bucket`.
- **Delete**: Failed in `cloud-launch-private-bucket` ("Access Denied").
- **List**: `cloudlaunch-visible-only-bucket9` listed, contents inaccessible ("Access Denied").

### CloudFront Distribution
- **Distribution**: `cloudlaunch-distribution`
- **Distribution ID**: `ECL5S6YSMT825`
- **Domain**: `d6bjvsbuq9pv.cloudfront.net`
- **Origin**: `cloud-launchsite-bucket` with OAC (`CloudLaunchOAC`).
- **WAF**: `CloudLaunchWAF` with AWS managed rules (Core rule set, Known bad inputs, IP reputation).
- **Test**: `index.html` served via CloudFront URL; WAF blocks malicious requests (e.g., `?param=1%20OR%201=1` returns 403 Forbidden).

## Task 2: VPC Design

### VPC Resources
- **VPC**: `cloudlaunch-vpc` (CIDR: `10.0.0.0/16`).
- **Subnets**:
  - `cloudlaunch-public-subnet` (`10.0.1.0/24`, `eu-north-1a`).
  - `cloudlaunch-app-subnet` (`10.0.2.0/24`, `eu-north-1b`).
  - `cloudlaunch-db-subnet` (`10.0.3.0/28`, `eu-north-1c`).
- **Internet Gateway**: `cloudlaunch-igw`, attached to `cloudlaunch-vpc`.
- **Route Tables**:
  - `cloudlaunch-public-rt`: Routes `0.0.0.0/0` to `cloudlaunch-igw`, associated with `cloudlaunch-public-subnet`.
  - `cloudlaunch-app-rt`: Private, associated with `cloudlaunch-app-subnet`.
  - `cloudlaunch-db-rt`: Private, associated with `cloudlaunch-db-subnet`.
- **Security Groups**:
  - `cloudlaunch-app-sg`: Allows HTTP (port 80) from `10.0.0.0/16`.
  - `cloudlaunch-db-sg`: Allows MySQL (port 3306) from `cloudlaunch-app-sg`.

### VPC Access Test
- Logged in as `cloudlaunch-user`.
- Viewed `cloudlaunch-vpc`, subnets, route tables, internet gateway, and security groups in VPC console.
- Modification attempts (e.g., create subnet) failed ("Access Denied"), confirming read-only access.

### Notes
- Resolved S3 "Access Denied" by updating `CloudLaunchS3Policy` and bucket policies.
- Fixed CloudFront "Access Denied" by correcting OAC and bucket policy.
- Addressed VPC access error by attaching `CloudLaunchVPCReadOnlyPolicy`.
