# User Guide: AI SOC Copilot

Welcome to AI SOC Copilot! This guide will walk you through how to use the platform to transform complex Wazuh security alerts into clear, actionable incident investigations.

---

## 1. Getting Started & Access

Because AI SOC Copilot is a web-based platform, there is no software to install on your computer.

1. Open your modern web browser (Google Chrome, Microsoft Edge, or Mozilla Firefox are recommended).
2. Navigate to your organization's AI SOC Copilot URL (e.g., `https://copilot.yourcompany.com`).
3. You will arrive immediately at the **Dashboard**.

---

## 2. Navigating the Platform

The application is designed to be simple and distraction-free. You will find the main navigation menu at the top of your screen:

- **Dashboard:** Your home screen. Here you can see a quick summary of recent investigations and start a new one.
- **New Investigation:** The area where you upload Wazuh alerts for the AI to analyze.
- **History:** A searchable ledger of all past investigations.
- **About:** Information about the current version of the software.

---

## 3. How to Start a New Investigation (Upload Workflow)

Starting an investigation takes only a few seconds.

1. Click on **New Investigation** in the top navigation bar, or click the big **Start New Investigation** button on the Dashboard.
2. You will see a large "Dropzone" box on the screen.
3. Locate the exported **Wazuh JSON alert** file on your computer.
4. **Drag and drop** the file into the Dropzone box. Alternatively, click the box to open your file browser and select the file.
5. Click **Start Investigation**.

> [!TIP]
> **File Requirements:** The system only accepts `.json` files. The maximum file size allowed is 10 MB.

---

## 4. Understanding the Investigation Results

Once you click "Start Investigation," you will see a progress tracker as the AI analyzes the alert. This usually takes between 10 to 30 seconds.

When complete, you will be taken to the **Investigation Results** screen. This screen is divided into several easy-to-read sections:

*   **Severity Badge:** A color-coded badge indicating how critical the threat is (Critical, High, Medium, Low).
*   **Executive Summary:** A quick 2-3 sentence overview of what happened.
*   **Plain-English Explanation:** A non-technical translation of the raw security data.
*   **MITRE ATT&CK:** Identifies the specific tactics and techniques the attacker is likely using.
*   **Indicators of Compromise (IoCs):** A list of suspicious IP addresses, file hashes, or domains found in the alert.
*   **Attack Narrative:** A timeline explaining how the attack likely unfolded.
*   **Recommendations:** Step-by-step instructions on how to stop the threat and fix the vulnerability.

> [!NOTE]
> Want to see the original alert data? Click the **View Raw JSON** accordion at the bottom of the page to verify the AI's findings against the raw logs.

---

## 5. Exporting an Incident Report

You do not need to manually copy and paste the findings into a document. The platform can do it for you!

1. On the **Investigation Results** screen, look for the Export buttons at the top right.
2. Click **Download PDF** to generate a professionally formatted incident report that is ready to hand to management or attach to an IT ticket.
3. Click **Download Markdown** if you wish to paste the formatted text into systems like Jira, GitHub, or internal wikis.

---

## 6. Reviewing Past Investigations

Need to find a report from last week?

1. Click **History** in the top navigation bar.
2. Use the **Search bar** to type in an IP address, rule ID, or keyword.
3. Use the **Filters** to narrow down results by Date or Severity.
4. Click on any row in the table to reopen that specific investigation and download its report again.

---

## 7. Troubleshooting

| Issue | How to Fix |
| :--- | :--- |
| **"Invalid File Format" Error** | You uploaded a file that is not a `.json` file (e.g., `.txt` or `.csv`). Export the alert from Wazuh specifically as JSON and try again. |
| **"File Too Large" Error** | The file you uploaded is over the 10 MB limit. Try exporting a single alert rather than a massive log dump. |
| **"AI Service Unavailable" Error** | The external AI engine is temporarily offline. Wait 60 seconds and click the "Retry" button. |
| **Missing IoCs or MITRE data** | If the AI outputs "Insufficient evidence," it means the raw alert did not contain enough data to make a definitive conclusion. This is a safety feature to prevent the AI from guessing. |

---

## 8. Frequently Asked Questions (FAQs)

**Q: Is the AI making autonomous changes to our firewalls or servers?**
**A:** No. AI SOC Copilot is strictly an advisory tool. It analyzes data and provides recommendations. A human analyst must always review the findings and take action.

**Q: Does the AI "hallucinate" or invent fake data?**
**A:** The system is heavily restricted to only analyze the data you upload. We enforce a "Confidence Score" (High, Medium, Low) on all results. If evidence is lacking, the AI is programmed to state that it cannot determine the answer rather than making one up.

**Q: How long does an investigation take?**
**A:** Most single-alert investigations are completed in under 30 seconds, depending on the complexity of the alert.

**Q: Can I delete an investigation?**
**A:** Yes. When viewing an investigation, click the **Delete** button at the top right of the screen. Please note that this action is permanent and cannot be undone.
