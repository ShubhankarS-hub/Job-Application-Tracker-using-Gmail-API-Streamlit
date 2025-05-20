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
![image](https://github.com/user-attachments/assets/0403c057-1e48-47ca-8b9e-a8f320d89dc4)

- **Filter by Status**: Select one or more application statuses to filter displayed data.
![image](https://github.com/user-attachments/assets/e5fde9e5-37e5-477a-8066-8b4c1697c6b3)

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


## 📄 Sample Output Dataset (`data_new_2.csv`)

| From            | Subject                        | Status     |
|-----------------|--------------------------------|------------|
| careers@xyz.com | Thank you for applying         | Applied    |
| hr@abc.com      | Unfortunately, you weren’t...  | Rejected   |
| jobs@pqr.com    | Interview invitation for...    | Interview  |

---

## 🖥️ Source Code

### 🔧 Python (Email Extraction + Classification)

<details>
<summary>Click to expand</summary>

```python
import os
import base64
import email
import pandas as pd

from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build

SCOPES = ['https://www.googleapis.com/auth/gmail.readonly']

def get_gmail_service():
    creds = None
    if os.path.exists('token.json'):
        creds = Credentials.from_authorized_user_file('token.json', SCOPES)
    else:
        flow = InstalledAppFlow.from_client_secrets_file('credentials.json', SCOPES)
        creds = flow.run_local_server(port=0)
        with open('token.json', 'w') as token:
            token.write(creds.to_json())
    return build('gmail', 'v1', credentials=creds)

def classify_email(text):
    text = text.lower()
    applied_phrases = ["thank you for applying", "your application has been received", "submitted successfully"]
    rejection_phrases = ["unfortunately", "we regret", "not selected"]
    interview_phrases = ["interview", "schedule a call", "invite"]

    found_applied = any(phrase in text for phrase in applied_phrases)
    found_rejected = any(phrase in text for phrase in rejection_phrases)
    found_interview = any(phrase in text for phrase in interview_phrases)

    if found_applied and found_rejected:
        return "Rejected"
    elif found_rejected:
        return "Rejected"
    elif found_applied or found_interview:
        return "Applied"

    return "Others"

def extract_emails_and_save(label_name='IMPORTANT'):
    service = get_gmail_service()
    label_id = [label['id'] for label in service.users().labels().list(userId='me').execute()['labels'] if label['name'].lower() == label_name.lower()][0]
    messages = []
    response = service.users().messages().list(userId='me', labelIds=[label_id], q="newer_than:90d").execute()
    messages.extend(response.get('messages', []))

    while 'nextPageToken' in response:
        response = service.users().messages().list(userId='me', labelIds=[label_id], q="newer_than:90d", pageToken=response['nextPageToken']).execute()
        messages.extend(response.get('messages', []))

    rows = []
    for msg in messages:
        msg_data = service.users().messages().get(userId='me', id=msg['id'], format='raw').execute()
        raw_msg = base64.urlsafe_b64decode(msg_data['raw'].encode('ASCII'))
        mime_msg = email.message_from_bytes(raw_msg)

        subject = mime_msg['subject'] or "No Subject"
        sender = mime_msg['from'] or "Unknown"
        body = ""

        if mime_msg.is_multipart():
            for part in mime_msg.walk():
                if part.get_content_type() == 'text/plain':
                    body += part.get_payload(decode=True).decode(errors='ignore')
        else:
            body = mime_msg.get_payload(decode=True).decode(errors='ignore')

        status = classify_email(subject + " " + body)

        rows.append({
            "From": sender,
            "Subject": subject,
            "Status": status
        })

    pd.DataFrame(rows).to_csv("data_new_2.csv", index=False)
    print("✅ Data saved to data_new_2.csv")

extract_emails_and_save()

---

*Made with ❤️ by Shubhankar Srivastava*
