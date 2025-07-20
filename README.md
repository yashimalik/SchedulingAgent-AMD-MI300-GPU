# 🤖 Agentic AI Scheduling Assistant - AMD MI300 GPU Hackathon

An intelligent meeting scheduling system powered by AMD MI300 GPU that uses AI agents to automate the complex process of coordinating meetings, extracting calendar events, and managing communications using the DeepSeek-7B LLM model.

## 🌟 Features

- **🔄 Intelligent Meeting Scheduling**: AI-powered agent that processes natural language meeting requests
- **📅 Calendar Integration**: Extracts and manages calendar events with conflict detection
- **🧠 LLM-Powered Processing**: Uses DeepSeek-7B model running on AMD MI300 GPU via vLLM
- **⚡ Real-time API**: Flask-based API endpoint for processing meeting requests
- **🌍 Multi-Timezone Support**: Handles timezone conflicts and finds optimal meeting times
- **📊 Smart Analytics**: Provides confidence scoring and meeting optimization

## 🏗️ Architecture

```
┌─────────────
```

## 🚀 Quick Start

### Prerequisites

- Python 3.9+
- OpenAI API key
- Optional: Google Calendar API credentials, Email credentials

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd amd_hackathon
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Set up environment variables**
   ```bash
   cp hackathon/.env.example .env
   # Edit .env with your API keys and configuration
   ```

4. **Run the demo**
   ```bash
   python -m hackathon.main demo
   ```

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# Required
OPENAI_API_KEY=your_openai_api_key_here
OPENAI_MODEL=gpt-4

# Optional - Calendar Integration
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret

# Optional - Email Notifications
EMAIL_ADDRESS=your_email@gmail.com
EMAIL_PASSWORD=your_app_password
SMTP_SERVER=smtp.gmail.com
SMTP_PORT=587

# Application Settings
DEBUG=True
LOG_LEVEL=INFO
DEFAULT_TIMEZONE=UTC
```

## 💻 Usage Examples

### 1. Demo Mode (Recommended for first run)
```bash
python -m hackathon.main demo
```
Runs a complete demonstration of the scheduling workflow with sample data.

### 2. Interactive Mode
```bash
python -m hackathon.main interactive
```
Interactive mode where you can input meeting details and participants.

### 3. Custom Meeting Scheduling
```bash
python -m hackathon.main custom
```
Schedules a predefined custom meeting (can be modified in the code).

### 4. Programmatic Usage

```python
import asyncio
from hackathon.workflow import SchedulingWorkflow
from hackathon.models.meeting import MeetingRequest, Participant, Priority
from datetime import datetime

async def schedule_meeting():
    # Create participants
    participants = [
        Participant(
            email="alice@company.com",
            name="Alice Johnson",
            timezone="US/Eastern",
            is_organizer=True
        ),
        Participant(
            email="bob@company.com", 
            name="Bob Smith",
            timezone="US/Pacific"
        )
    ]
    
    # Create meeting request
    meeting_request = MeetingRequest(
        title="Project Kickoff",
        description="Initial project planning meeting",
        participants=participants,
        duration_minutes=90,
        preferred_time_slots=[],
        priority=Priority.HIGH,
        created_at=datetime.now(),
        organizer_email="alice@company.com"
    )
    
    # Create scheduling context
    from hackathon.models.meeting import SchedulingContext
    context = SchedulingContext(
        user_preferences={},
        timezone_preferences={p.email: p.timezone for p in participants},
        working_hours={},
        calendar_permissions={p.email: True for p in participants},
        communication_preferences={p.email: "email" for p in participants}
    )
    
    # Schedule the meeting
    workflow = SchedulingWorkflow()
    result = await workflow.schedule_meeting(meeting_request, context)
    
    if result.success:
        print(f"Meeting scheduled: {result.meeting.scheduled_time.start_time}")
    else:
        print(f"Scheduling failed: {result.error_message}")

# Run the scheduling
asyncio.run(schedule_meeting())
```

## 🔧 Core Components

### Scheduler Agent (`hackathon/agents/scheduler_agent.py`)
- **Purpose**: Main orchestrator that coordinates the entire scheduling process
- **Technology**: LangGraph state machine with conditional edges
- **Capabilities**:
  - Analyzes meeting requests using AI
  - Orchestrates workflow between agents
  - Handles conflict resolution
  - Makes final scheduling decisions

### Calendar Agent (`hackathon/agents/calendar_agent.py`)
- **Purpose**: Manages calendar integrations and availability scanning
- **Capabilities**:
  - Scans participant calendars (Google Calendar, Outlook)
  - Calculates availability windows
  - Finds common free time slots
  - Creates and cancels calendar events
  - Handles timezone conversions

### Communicator Agent (`hackathon/agents/communicator_agent.py`)
- **Purpose**: Handles all communications and notifications
- **Capabilities**:
  - Sends meeting invitations
  - Provides meeting confirmations
  - Handles reschedule requests
  - Sends conflict notifications
  - Manages meeting reminders
  - Supports multiple communication channels (email, future: SMS, push)

### Data Models (`hackathon/models/meeting.py`)
- `MeetingRequest`: Incoming meeting scheduling request
- `Meeting`: Complete meeting information with status
- `Participant`: Meeting participant details
- `TimeSlot`: Time slot with timezone and availability
- `SchedulingContext`: User preferences and configuration

## 🎯 Key Features Deep Dive

### 1. Multi-Timezone Intelligence
- Automatically detects and handles timezone conflicts
- Finds optimal meeting times across multiple timezones
- Provides timezone-aware scheduling suggestions
- Displays times in each participant's local timezone

### 2. AI-Powered Conflict Resolution
- Uses GPT-4 to analyze scheduling conflicts
- Generates intelligent alternatives
- Negotiates between participant preferences
- Provides contextual explanations for scheduling decisions

### 3. Extensible Calendar Integration
- Plugin architecture for different calendar providers
- Currently supports simulation mode with Google Calendar hooks
- Easy to extend for Outlook, Office 365, and other providers

### 4. Smart Communication
- Context-aware message generation
- Professional email templates
- Handles various notification types (invitations, confirmations, reschedules)
- Customizable communication preferences per participant

### 5. Workflow Orchestration
- Built on LangGraph for robust state management
- Handles errors and retries gracefully
- Supports complex workflow branching
- Provides detailed execution results

## 🛠️ Development

### Project Structure
```
hackathon/
├── agents/                 # AI agents
│   ├── scheduler_agent.py  # Main orchestrator
│   ├── calendar_agent.py   # Calendar operations
│   └── communicator_agent.py # Communications
├── models/                 # Data models
│   └── meeting.py          # Meeting-related data structures
├── utils/                  # Utilities
│   └── timezone_utils.py   # Timezone handling
├── config.py              # Configuration management
├── workflow.py            # Main workflow orchestration
└── main.py               # Entry point and examples
```

### Adding New Features

1. **New Agent**: Create a new agent class in `agents/`
2. **New Model**: Add data structures to `models/`
3. **New Integration**: Extend agents with new service integrations
4. **New Workflow**: Modify `workflow.py` to add new orchestration paths

### Testing

Run the demo mode to test all components:
```bash
python -m hackathon.main demo
```

Check logs for detailed execution information.

## 🔮 Future Enhancements

- **🔗 Additional Calendar Integrations**: Office 365, CalDAV, iCal
- **📱 Mobile Notifications**: SMS, push notifications
- **🧠 Advanced AI Features**: 
  - Meeting preference learning
  - Predictive scheduling
  - Natural language processing for meeting requests
- **🔄 Recurring Meetings**: Support for recurring meeting patterns
- **📊 Analytics Dashboard**: Meeting scheduling analytics and insights
- **🌐 Web Interface**: Browser-based scheduling interface
- **🤝 Meeting Room Integration**: Physical meeting room booking
- **👥 Team Optimization**: AI-powered team meeting optimization

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🆘 Support

For questions, issues, or contributions:
- Create an issue in the repository
- Check the logs when running in debug mode
- Review the architecture documentation

## 🎉 Acknowledgments

- Built with [LangGraph](https://github.com/langchain-ai/langgraph) for workflow orchestration
- Powered by [OpenAI GPT-4](https://openai.com/) for intelligent decision making
- Uses [LangChain](https://github.com/langchain-ai/langchain) for AI integration

---
