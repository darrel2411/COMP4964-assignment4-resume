# 🧑‍💻 Assignment 4 – CI/CD Pipeline Documentation

**Author:** Darrel Tapilaha  
**Course:** COMP 4964 – DevOps  
**Project:** Resume Website Deployment on using AWS CodePipeline

---

## 🧩 1. Architecture Overview

### Diagram

![Architecture Diagram](/COMP%204964_assignment4_diagram.png)

### Workflow Summary

1. The source code (HTML, CSS, assets, PDF resume) is stored in a GitHub repository.
2. **AWS CodePipeline** detects any new commit pushed to the `main` branch.
3. **CodePipeline** automatically triggers a **CodeBuild** project.
4. **CodeBuild** runs the commands in `buildspec.yml` to synchronize website files to the **S3** bucket.
5. The **S3 bucket** hosts the website publicly as a static site.

---

## ☁️ 2. AWS Services Used

| Service                         | Purpose                                                                      |
| ------------------------------- | ---------------------------------------------------------------------------- |
| **S3 (Simple Storage Service)** | Hosts the static website files (HTML/CSS/PDF).                               |
| **CodePipeline**                | Automates CI/CD workflow — detects repo updates and triggers builds.         |
| **CodeBuild**                   | Executes `aws s3 sync` to deploy updated files to the S3 bucket.             |
| **IAM**                         | Provides permissions for CodePipeline and CodeBuild to access GitHub and S3. |
| **GitHub**                      | Stores and versions the website source code.                                 |

---

## ⚙️ 3. Project Setup Steps

### Step 1 — Create the S3 Bucket

1. Go to **S3 → Create bucket**
2. Name the bucket uniquely, e.g. `assignment4-resume`
3. Region: `us-west-2`
4. Disable **Block all public access**
5. In **Properties → Static website hosting**
   - Enable “Host a static website”
   - Set `index.html` as the Index document
6. Add a **Bucket Policy** for public read access:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "PublicReadGetObject",
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::assignment4-resume/*"
       }
     ]
   }
   ```

### Step 2 — Prepare the Repository

**Project structure:**

```
COMP4964-Assignment4-Resume/
├─ assets/
│  ├─ Resume Darrel Tapilaha.pdf
├─ index.html
├─ style.css
├─ buildspec.yml
└─ README.md
```

### Step 3 — Create the AWS CodeBuild project

1. Project name: resume-site
2. Source: GitHub
3. Environment:
   - OS: Amazon Linux 2
   - Runtime: Standard
   - Image: aws/codebuild/amazonlinux2-x86_64-standard:5.0
4. Buildspec: use buildspec.yml
5. Service role: Create new → attachh S3 permission:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "List",
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::<your-s3-bucket>"
    },
    {
      "Sid": "Objects",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"],
      "Resource": "arn:aws:s3:::<your-s3-bucket>/*"
    }
  ]
}
```

### Step 4 — Create the AWS CodePipeline

1. Pipeline name: resume-site-pipeline
2. Source provider: GitHub (Version 2)
   - Repo: <github_username>/<project_repository>
   - Branch: main
3. Build provider: AWS CodeBuild (resume-site)
4. Deploy stage: Skipped (buildspec handles S3 sync)
5. Review and create pipeline.

### Step 5 — Test the Pipeline

1. Edit index.html
2. Commit and push
3. CodePipeline runs automatically → triggers CodeBuild → updates S3.
4. Refresh the static website URL to confirm update.

## 🧠 3. Troubleshooting

| Problem                     | Possible Cause                                 | Fix                                                                     |
| --------------------------- | ---------------------------------------------- | ----------------------------------------------------------------------- |
| **YAML_FILE_ERROR**         | Wrong indentation or tabs in `buildspec.yml`   | Replace tabs with spaces and validate YAML syntax.                      |
| **Access Denied (403)**     | Public access blocked or missing bucket policy | Disable “Block all public access” and apply the public-read S3 policy.  |
| **Build failed (exit 252)** | Incorrect `aws s3 sync` syntax                 | Remove extra `\` or quotes; use a single-line command.                  |
| **No updates on S3**        | Cached files or wrong bucket name              | Ensure the `--delete` flag is present and verify `WEBSITE_BUCKET` name. |

## 4. 🧹 Clean Up

To avoid unnecessary AWS charges:

1. Delete the CodePipeline and CodeBuild projects.
2. Empty and delete the S3 bucket.
3. Delete the IAM roles if not reused.
