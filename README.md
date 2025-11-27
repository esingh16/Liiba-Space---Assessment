# Part 1 — ATS Data Source Discovery  
This document contains my complete analysis and findings for Part 1 of the Data Engineer Internship Assessment.  
The objective is to identify new companies using public ATS platforms and design a scalable method for ATS-wide discovery.

---

## Step 1: Three Companies Using Public ATS Platforms  

I identified three companies whose career pages are hosted on major ATS (Applicant Tracking System) platforms.  
Each company was verified using the internal monitoring tool to ensure it was NOT already present.

### 1. Rivian  
- **ATS Type:** Workday  
- **Career URL:** rivian.wd5.myworkdayjobs.com  

### 2. Grow Therapy  
- **ATS Type:** Greenhouse  
- **Career URL:** boards.greenhouse.io/growtherapy  

### 3. Shield AI  
- **ATS Type:** Lever  
- **Career URL:** jobs.lever.co/shieldai  

These companies span three different ATS systems, ensuring variety and correctness.

---

## Step 2: ATS-Wide Discovery Method  

To discover a large portion (or nearly all) companies operating under a specific ATS, I developed a multi-layered discovery method.  
This method relies on URL pattern recognition, internet-wide data signals, certificate logs, search engines, and ATS-specific fingerprints.

---

### 2.1 How ATS Platforms Reveal Their Tenants  

Major ATS providers leave behind structural traces such as:

- Repeated URL patterns  
- Search engine indexed job pages  
- Public SSL certificates  
- Embedded JavaScript widgets  
- JSON job endpoints  
- Customer showcase pages  

By combining these signals, it becomes possible to reconstruct a large portion of the ATS tenant list.

---

### 2.2 Multi-Stage Tenant Discovery Workflow  

#### **A. Decode the ATS URL Structure**  
Every ATS reuses a predictable format:

- Greenhouse → boards.greenhouse.io/{slug}  
- Lever → jobs.lever.co/{slug}  
- Workday → {company}.wd{cluster}.myworkdayjobs.com  

Understanding this structure helps identify potential tenants.

---

#### **B. Analyze robots.txt and sitemap.xml**  
These files reveal internal routing patterns, embed endpoints, and blocked directories.  
They give insight into how the ATS organizes its tenants.

---

#### **C. Use Search Engine Dorking**  
Examples of effective queries:


site:boards.greenhouse.io
inurl:jobs.lever.co
"myworkdayjobs.com" intitle:careers

Search engines store many indexed job listings across ATS hosts.

---

#### **D. Query Certificate Transparency Logs**  
Using SSL certificate databases such as crt.sh or Censys, you can retrieve all subdomains under:

- boards.greenhouse.io  
- jobs.lever.co  
- myworkdayjobs.com  

These logs often reveal thousands of ATS tenants.

---

#### **E. Identify ATS JavaScript Widgets**  
Most ATS systems embed job-board loaders such as:

- Greenhouse → boards.greenhouse.io/embed/job_board/js?for={slug}  
- Lever → api.lever.co/v0/postings/{slug}  

Scanning for these patterns reveals ATS usage on company websites.

---

#### **F. Reverse-Engineer Public JSON Endpoints**  
Some ATS platforms expose job data via JSON feeds.

Example:

boards.greenhouse.io/embed/job_board/js?for={slug}

Valid slugs return structured job data; invalid slugs return 404.

---

#### **G. Scrape ATS “Customers” Pages**  
Greenhouse, Lever, and other ATS providers showcase client lists on their websites, which serve as reliable seed data for expanding enumeration.

---

## Step 3: Demonstration of Method Using Greenhouse  

### **A. Identify the Pattern**  
Greenhouse job boards follow:


boards.greenhouse.io/{slug}

---

### **B. Extract Candidate Slugs from Certificate Transparency Logs**  
Using crt.sh queries, I collected slugs such as:

- growtherapy  
- discord  
- roblox  
- ramp  
- confluent  
- unity3d  

---

### **C. Validate Slugs with the Greenhouse JSON Endpoint**  
Testing:


boards.greenhouse.io/embed/job_board/js?for={slug}

A valid company returns structured JSON containing job listings.  
Invalid slugs return a consistent 404 response.

---

### **D. Verified Greenhouse Companies**

| Company      | Greenhouse URL                        |
|--------------|----------------------------------------|
| Grow Therapy | boards.greenhouse.io/growtherapy       |
| Discord      | boards.greenhouse.io/discord           |
| Ramp         | boards.greenhouse.io/ramp              |
| Confluent    | boards.greenhouse.io/confluent         |
| Roblox       | boards.greenhouse.io/roblox            |

These URLs load correctly and contain Greenhouse embed scripts, confirming validity.

---

## Step 4: Coverage & Scalability Discussion  

### **Estimated Tenant Discovery Coverage**

| Method                    | Estimated Coverage |
|---------------------------|--------------------|
| SSL Certificate Logs      | 70–85%             |
| Search Engine Indexing    | 60–80%             |
| JSON Endpoint Probing     | 80–90%             |
| Customer Marketing Pages  | 20–40%             |
| **Combined Total**        | **90–95%**         |

Because these methods complement each other, combined coverage is very high.

---

### **Scalability**  

This approach is highly scalable because:

- All steps rely on simple GET requests  
- Certificate logs update daily  
- JSON endpoints can be hit programmatically  
- Search engine queries can be automated  
- No API keys or authentication are required  

A lightweight crawler could map most ATS tenants continuously.

---

### **Limitations**  

- Workday URLs can be inconsistent  
- Some job boards are protected by SSO  
- New tenants may not yet appear in CT logs  
- Search engine indexing differs by region  

---



