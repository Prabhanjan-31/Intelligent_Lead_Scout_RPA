# IntelligentLeadScout

### Automated Web Scraping, Scoring, and Dispatch Pipeline

IntelligentLeadScout is a Robotic Process Automation (RPA) solution built in UiPath. It acts as an autonomous agent that searches for local businesses on Justdial, scrapes unstructured directory data, normalizes it into a structured Excel database, calculates a custom mathematical "Leadscore" for each business, and dispatches the final analytical report via Gmail.

---

## Architecture & Workflows

The project is modularized into several specific sequences to ensure clean data handoffs and maintainability.

| Workflow File | Core Functionality |
| :--- | :--- |
| **`Main.xaml`** | The primary entry point of the project. It directly invokes the search sequence to begin the automation lifecycle. |
| **`SearchJustdial.xaml`** | Prompts the user for runtime inputs (**Business Type** and **Location**). It then opens Google Chrome, navigates to Justdial, enters the search query, and utilizes automated keystrokes (`PgDn`) to force lazy-loading of business listings. |
| **`ScrapeDetails.xaml`** | The core extraction engine. It identifies listing containers (`*resultbox_info*`) and uses Regular Expressions (Regex) to cleanly extract the Name, Address, Rating (`[0-5]\.[0-9]`), Phone (`\d{10,12}`), and Review Count. It calculates the Leadscore, writes the structured data to `Data.xlsx`, and triggers the AI analysis. |
| **`AnalyzeLeads_GPT.xaml`** | *Referenced Workflow.* Takes the structured data table and passes it to an AI model for qualitative analysis and recommendation generation. |
| **`Monitoring email.xaml`** | The final distribution layer. It prompts the user for a destination Email ID, opens the Gmail web interface, drafts a new email with the subject **"Business suggestion with Data file"**, inserts the AI's recommendation into the body, attaches `Data.xlsx`, and automates the send click. |

---

## The Leadscore Algorithm

To objectively rank the extracted businesses, `ScrapeDetails.xaml` applies a weighted logarithmic formula to balance the raw star rating against the total volume of reviews.

$Leadscore = Round((Rating * 0.7) + (log_10(Reviews + 1) * 0.3), 2)$

* **Rating Weight (70%):** Prioritizes the actual customer satisfaction score.
* **Review Volume Weight (30%):** Applies a Base-10 logarithm to the review count. This provides a credibility bonus to businesses with high interaction volumes while preventing extreme review counts from disproportionately dominating the score.

---

## Technical Specifications

### Data Schema
The robot dynamically builds a `DataTable` containing the following attributes for each extracted lead:
* Name
* Rating
* Reviews
* Address
* Phone
* Maplink (Dynamically generated Google Maps URL)
* Leadscore (Calculated Double)

### Dependencies
According to `project.json`, this project relies on the following UiPath activity packages:
* `UiPath.Excel.Activities` [2.11.4]
* `UiPath.Mail.Activities` [1.12.3]
* `UiPath.PDF.Activities` [3.16.0]
* `UiPath.System.Activities` [21.10.6]
* `UiPath.UIAutomation.Activities` [21.10.8]
* `UiPath.WebAPI.Activities` [1.9.2]
* `ExceltoPDFConversionActivities` [1.1.5]

---

## Execution Guide

1.  Open the project in **UiPath Studio** (Version 21.10.10.0 or higher recommended).
2.  Ensure all dependencies are restored successfully.
3.  Run `Main.xaml`.
4.  Provide the requested **Business Type** (e.g., "Restaurants") and **Location** (e.g., "Bangalore") in the interactive pop-ups.
5.  Allow the robot to control the mouse and keyboard to extract the data.
6.  Provide the destination **Email ID** when prompted to receive the final report and `Data.xlsx` file.
