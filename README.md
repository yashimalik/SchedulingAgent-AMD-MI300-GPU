# SchedulingAgent-AMD-MI300-GPU

A meeting scheduling agent powered by AMD MI300 GPU using DeepSeek-7B LLM for intelligent calendar management and meeting coordination.

## Overview

This project implements an AI-powered scheduling assistant that processes meeting requests in JSON format and returns optimized calendar events. The system leverages vLLM running on AMD MI300 GPU hardware to provide fast, intelligent meeting scheduling capabilities.

## Repository Contents

- `Scheduler_Agent.ipynb` - Main scheduling agent implementation
- `Calendar_Event_Extraction.ipynb` - Calendar event processing logic
- `Sample_AI_Agent.ipynb` - Basic AI agent example and setup
- `Input_Output_Formats.ipynb` - Input/output format specifications
- `findFreeSlot.ipynb` - Free time slot detection algorithm
- `Cred_to_Token.ipynb` - Authentication token management
- `GoogleAuth.md` - Google Calendar authentication guide
- `AI_Scheduling_Assistant - ppt.pptx` - Project presentation
- `errors.log` - System error logs

## Setup Instructions

### 1. vLLM Server Configuration

Launch the vLLM server with Meta-Llama-3.1-8B-Instruct model:

HIP_VISIBLE_DEVICES=0 vllm serve /home/user/Models/meta-llama/Meta-Llama-3.1-8B-Instruct \
        --gpu-memory-utilization 0.9 \
        --swap-space 16 \
        --disable-log-requests \
        --dtype float16 \
        --max-model-len 8192 \
        --tensor-parallel-size 1 \
        --host 0.0.0.0 \
        --port 4000 \
        --num-scheduler-steps 10 \
        --max-num-seqs 128 \
        --max-num-batched-tokens 8192 \
        --distributed-executor-backend "mp"

### 2. AI Agent Implementation

The core AI agent processes email content to extract meeting information:

```python
class AI_AGENT:
    def __init__(self, client, MODEL_PATH):
        self.base_url = BASE_URL
        self.model_path = MODEL_PATH

    def parse_email(self, email_text):
        response = client.chat.completions.create(
            model=self.model_path,
            temperature=0.0,
            messages=[{
                "role": "user",
                "content": f"""
                You are an Agent that helps in scheduling meetings.
                Your job is to extract Email ID's and Meeting Duration.
                You should return:
                1. List of email id's of participants (comma-separated).
                2. Meeting duration in minutes.
                3. Time constraints (e.g., 'next week').
                If the List of email id's of participants are just names, then append @amd.com at the end and return. 
                Return as json with 'participants', 'time_constraints' & 'meeting_duration'.
                Strictly follow the instructions. Strictly return dict with participants email id's, time constraints & meeting duration in minutes only. 
                Do not add any other instructions or information. 
                
                Email: {email_text}
                """
            }]
        )
        return json.loads(response.choices[0].message.content)
```

## Input Format

The system accepts meeting requests in the following JSON structure:

```json
{
    "Request_id": "6118b54f-907b-4451-8d48-dd13d76033a5",
    "Datetime": "19-07-2025T12:34:55",
    "Location": "IISc Bangalore",
    "From": "userone.amd@gmail.com",
    "Attendees": [
        {
            "email": "usertwo.amd@gmail.com"
        },
        {
            "email": "userthree.amd@gmail.com"
        }
    ],
    "Subject": "Agentic AI Project Status Update",
    "EmailContent": "Hi team, let's meet on Thursday for 30 minutes to discuss the status of Agentic AI Project."
}
```

## Output Format

The system returns scheduled events in this JSON structure:

```json
{
    "Request_id": "6118b54f-907b-4451-8d48-dd13d76033a5",
    "Datetime": "19-07-2025T12:34:55",
    "Location": "IISc Bangalore",
    "From": "userone.amd@gmail.com",
    "Attendees": [
        {
            "email": "userone.amd@gmail.com",
            "events": [
                {
                    "StartTime": "2025-07-24T10:30:00+05:30",
                    "EndTime": "2025-07-24T11:00:00+05:30",
                    "NumAttendees": 3,
                    "Attendees": [
                        "userone.amd@gmail.com",
                        "usertwo.amd@gmail.com",
                        "userthree.amd@gmail.com"
                    ],
                    "Summary": "Agentic AI Project Status Update"
                }
            ]
        },
        {
            "email": "usertwo.amd@gmail.com",
            "events": [
                {
                    "StartTime": "2025-07-24T10:00:00+05:30",
                    "EndTime": "2025-07-24T10:30:00+05:30",
                    "NumAttendees": 3,
                    "Attendees": [
                        "userone.amd@gmail.com",
                        "usertwo.amd@gmail.com",
                        "userthree.amd@gmail.com"
                    ],
                    "Summary": "Team Meet"
                },
                {
                    "StartTime": "2025-07-24T10:30:00+05:30",
                    "EndTime": "2025-07-24T11:00:00+05:30",
                    "NumAttendees": 3,
                    "Attendees": [
                        "userone.amd@gmail.com",
                        "usertwo.amd@gmail.com",
                        "userthree.amd@gmail.com"
                    ],
                    "Summary": "Agentic AI Project Status Update"
                }
            ]
        },
        {
            "email": "userthree.amd@gmail.com",
            "events": [
                {
                    "StartTime": "2025-07-24T10:00:00+05:30",
                    "EndTime": "2025-07-24T10:30:00+05:30",
                    "NumAttendees": 3,
                    "Attendees": [
                        "userone.amd@gmail.com",
                        "usertwo.amd@gmail.com",
                        "userthree.amd@gmail.com"
                    ],
                    "Summary": "Team Meet"
                },
                {
                    "StartTime": "2025-07-24T13:00:00+05:30",
                    "EndTime": "2025-07-24T14:00:00+05:30",
                    "NumAttendees": 1,
                    "Attendees": [
                        "SELF"
                    ],
                    "Summary": "Lunch with Customers"
                },
                {
                    "StartTime": "2025-07-24T10:30:00+05:30",
                    "EndTime": "2025-07-24T11:00:00+05:30",
                    "NumAttendees": 3,
                    "Attendees": [
                        "userone.amd@gmail.com",
                        "usertwo.amd@gmail.com",
                        "userthree.amd@gmail.com"
                    ],
                    "Summary": "Agentic AI Project Status Update"
                }
            ]
        }
    ],
    "Subject": "Agentic AI Project Status Update",
    "EmailContent": "Hi team, let's meet on Thursday for 30 minutes to discuss the status of Agentic AI Project.",
    "EventStart": "2025-07-24T10:30:00+05:30",
    "EventEnd": "2025-07-24T11:00:00+05:30",
    "Duration_mins": "30",
    "MetaData": {}
}
```

## API Implementation

The main function `your_meeting_assistant()` processes meeting requests:

```python
def your_meeting_assistant(data):
    # Takes Meeting request JSON as Input
    # Returns processed JSON with scheduled events
    # Implementation details in Scheduler_Agent.ipynb
    pass
```

## API Server

The system runs a Flask server on port 5000:

```python
@app.route('/receive', methods=['POST'])
def receive():
    data = request.get_json()
    new_data = your_meeting_assistant(data)
    return jsonify(new_data)

app.run(host='0.0.0.0', port=5000)
```

## Testing

Test the API by sending JSON requests:

```python
import requests
import json

SERVER_URL = "<YOUR IP ADDRESS>"
INPUT_JSON_FILE = "JSON_Samples/Input_Request.json"

with open(INPUT_JSON_FILE) as f:
    input_json = json.load(f)

response = requests.post(SERVER_URL+":5000/receive", json=input_json, timeout=10)
print(response.json())
```

## Notebooks

- **Sample_AI_Agent.ipynb**: Demonstrates basic AI agent setup with vLLM and OpenAI API communication
- **Scheduler_Agent.ipynb**: Complete scheduling agent implementation
- **Calendar_Event_Extraction.ipynb**: Calendar integration and event processing
- **findFreeSlot.ipynb**: Algorithm for finding available meeting slots
- **Input_Output_Formats.ipynb**: Detailed format specifications and examples

## Authentication

Google Calendar integration requires proper authentication setup. See `GoogleAuth.md` for detailed instructions and use the credential notebooks for token management.

## Evaluation Criteria

The system is evaluated on:
- **Correctness of Output**: Accuracy and precision of results
- **Roundtrip Latency**: Speed and efficiency of processing and response
- **GitHub Repository Maintenance**: Code organization, documentation, and commit hygiene
- **Creativeness in Approach**: Innovation, originality, and problem-solving uniqueness
- **Performance-Based Scoring**: Combined assessment of correctness, latency, repository quality, and creativity

## Requirements

- AMD MI300 GPU
- vLLM server
- DeepSeek-7B chat model
- Python with required dependencies
- Flask for API server
- Google Calendar API (optional)

## Usage

1. Start the vLLM server with the provided configuration
2. Run the Flask API server
3. Send meeting request JSON to the `/receive` endpoint
4. Receive processed scheduling results in the specified output format

The system processes natural language meeting requests and returns optimized calendar events with conflict resolution and intelligent time slot allocation.
