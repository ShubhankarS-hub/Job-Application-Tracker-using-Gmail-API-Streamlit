# Job-Application-Email-Classifier

## Project Overview

This project connects to your Gmail account using the **Gmail API** with **OAuth2 authentication** via **Google Cloud Platform (GCP)**, fetches emails labeled as **Important** from the last 90 days, and classifies them into job application statuses: **Applied**, **Rejected**, **Interview**, or **Others**.

The classified data is saved to a CSV file and visualized in an interactive **Streamlit** dashboard, allowing you to track your job application progress effortlessly.

## Dataset

The processed dataset (`data_new_2.csv`) contains the following columns:
- **From**: Sender’s email address.
- **Subject**: Email subject line.
- **Status**: Email classification based on content (`Applied`, `Rejected`, `Interview`, or `Others`).

## Functionality

1. **Gmail API Authentication**: Uses OAuth2 credentials to securely access Gmail with read-only permissions.
2. **Email Extraction**: Fetches emails from the Important label, limited to the last 90 days.
3. **Email Classification**: Classifies emails by searching for key phrases in the subject and body to determine status.
4. **CSV Export**: Saves the classified emails as a CSV file (`data_new_2.csv`).
5. **Streamlit Dashboard**: Loads the CSV and visualizes job application statuses, summary metrics, and charts with interactive filtering.

## How to Run

### Prerequisites

- Python 3.7+
- A Google Cloud Platform project with Gmail API enabled.
- OAuth2 credentials JSON file (`credentials.json`) from GCP.

### Setup

1. Install required packages:
    ```bash
    pip install --upgrade google-api-python-client google-auth google-auth-oauthlib pandas streamlit plotly matplotlib
    ```
2. Place `credentials.json` in your project directory.

3. Run the email extraction script:
    ```bash
    python extract_emails.py
    ```
    This will prompt you to authenticate via browser on first run and create `token.json`.

4. Run the Streamlit dashboard:
    ```bash
    streamlit run app.py
    ```

## Dashboard Features

- **Summary Metrics**: Total applications and rejections count.
- **Filter by Status**: Select one or more application statuses to filter displayed data.
- **Data Table**: View detailed email records with sender, subject, and status.
- **Visualizations**:
  - Bar chart showing counts by status.
![image](https://github.com/user-attachments/assets/e1a199ae-6baa-4660-8829-12b3e6e89c47)

  - Pie chart for application status distribution.
![image](https://github.com/user-attachments/assets/154b4378-386a-4ece-9193-f9cfb886eace)

## Libraries Used

- **google-api-python-client**, **google-auth**, **google-auth-oauthlib** — Gmail API and OAuth2 authentication.
- **pandas** — Data handling and CSV operations.
- **streamlit** — Dashboard UI.
- **plotly.express**, **matplotlib.pyplot** — Data visualization.

## Notes

- The classification logic is keyword-based and may be refined for better accuracy.
- Make sure to keep your `credentials.json` and `token.json` files secure and do not commit them to public repositories.



---

*Made with ❤️ by Shubhankar Srivastava*
