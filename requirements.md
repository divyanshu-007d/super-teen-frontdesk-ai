# Requirements Document

## Introduction

This document specifies the requirements for an AI-powered hospital front desk system that replaces traditional physical front desks with a voice-first, multi-channel digital assistant. The system handles administrative and operational tasks only, explicitly excluding all clinical functions such as medical diagnosis, treatment suggestions, or clinical decision-making. The system addresses long queues, manual workflows, limited staff capacity, and accessibility challenges in Indian hospitals by providing patient self-registration, appointment booking, queue token generation, and hospital navigation across multiple channels (QR code/web, phone call, SMS, and hospital kiosk).

## Glossary

- **AI_Assistant**: The conversational AI component that interacts with patients through voice and text
- **Patient**: An individual seeking hospital services who interacts with the system
- **Registration_Service**: The component that collects patient identity information and creates patient records
- **Booking_Service**: The component that handles appointment scheduling and queue token generation
- **ERP_Integration**: The interface that connects the system to the hospital's existing Enterprise Resource Planning system
- **Session_Manager**: The component that maintains conversation state across interactions
- **Receipt_Generator**: The component that creates PDF and printed receipts with patient information
- **Navigation_Service**: The component that provides directions to hospital departments
- **OTP_Verifier**: The component that validates one-time passwords sent to patient mobile numbers
- **Intent_Classifier**: The AI component that determines patient intent from natural language input
- **Speech_Service**: The component that handles speech-to-text and text-to-speech conversion
- **Channel**: A method of system access (QR code/web, phone call, SMS, or kiosk)
- **Queue_Token**: A numbered ticket that assigns a patient's position in a department queue
- **Appointment**: A scheduled time slot for a patient to visit a specific department or doctor
- **Clinical_Boundary**: The rule-based system that prevents the AI from providing medical advice
- **Hospital_Admin**: A hospital staff member with access to system configuration and analytics
- **Multi_Hospital_Instance**: A deployment configuration supporting multiple hospitals or hospital chains
- **Audit_Log**: A tamper-proof record of all system transactions and patient interactions

## Requirements

### Requirement 1: Patient Self-Registration

**User Story:** As a patient, I want to register myself with the hospital, so that I can access hospital services without waiting in physical queues.

#### Acceptance Criteria

1. WHEN a patient initiates registration through any channel, THE Registration_Service SHALL collect the patient's name, mobile number, and age
2. WHERE OTP verification is enabled, WHEN a patient provides their mobile number, THE OTP_Verifier SHALL send a one-time password and validate it before proceeding
3. WHEN a patient completes registration, THE ERP_Integration SHALL create a patient record in the hospital's ERP system
4. WHEN registration is successful, THE Receipt_Generator SHALL create a confirmation with the patient's registration details
5. IF a patient provides incomplete information, THEN THE AI_Assistant SHALL prompt for the missing required fields
6. THE Registration_Service SHALL NOT collect or store medical history, symptoms, or clinical information during registration

### Requirement 2: Appointment Booking and Queue Token Generation

**User Story:** As a patient, I want to book an appointment or get a queue token, so that I can plan my hospital visit efficiently.

#### Acceptance Criteria

1. WHEN a patient requests an appointment, THE AI_Assistant SHALL collect the department or service name and preferred date/time
2. WHEN a patient requests a queue token, THE Booking_Service SHALL generate a numbered token for the selected department
3. WHEN an appointment is booked, THE Booking_Service SHALL verify availability and confirm the time slot
4. WHEN a booking is confirmed, THE Receipt_Generator SHALL create a receipt containing patient name, token or appointment ID, department, date/time, and navigation instructions
5. WHEN a booking is confirmed, THE system SHALL send an SMS confirmation to the patient's mobile number
6. IF the requested time slot is unavailable, THEN THE AI_Assistant SHALL suggest alternative available slots
7. THE Booking_Service SHALL persist all bookings to the database immediately

### Requirement 3: Multi-Channel Access

**User Story:** As a patient, I want to access the system through multiple channels, so that I can use the method most convenient for me.

#### Acceptance Criteria

1. THE system SHALL provide access through QR code (mobile browser), phone call, SMS, and hospital kiosk channels
2. WHEN a patient scans a QR code, THE system SHALL open a Next.js web interface with voice and text interaction capabilities
3. WHEN a patient calls the hospital phone number, THE Speech_Service SHALL handle voice interaction without requiring app installation
4. WHEN a patient sends an SMS, THE system SHALL process text-based booking and registration requests
5. WHEN a patient uses a hospital kiosk, THE system SHALL provide a tablet interface with voice-first UX and receipt printing capability
6. FOR ALL channels, THE Session_Manager SHALL maintain conversation state throughout the interaction
7. FOR ALL channels, THE system SHALL provide consistent functionality and user experience

### Requirement 4: Hospital Navigation and Directions

**User Story:** As a patient, I want to receive directions to hospital departments, so that I can easily find where I need to go.

#### Acceptance Criteria

1. WHEN a patient requests directions, THE Navigation_Service SHALL provide step-by-step instructions to the requested department
2. WHEN a booking is confirmed, THE Receipt_Generator SHALL include navigation instructions in the receipt
3. THE Navigation_Service SHALL support queries for departments, services, doctors, and facilities
4. THE Navigation_Service SHALL provide directions appropriate to the patient's current location (entrance, kiosk location, etc.)

### Requirement 5: Clinical Boundary Enforcement

**User Story:** As a hospital administrator, I want the AI to refuse all clinical requests, so that the system operates safely within non-clinical boundaries.

#### Acceptance Criteria

1. WHEN a patient asks for medical diagnosis, THE Clinical_Boundary SHALL detect the request and THE AI_Assistant SHALL refuse with an explanation
2. WHEN a patient asks for medicine suggestions, THE Clinical_Boundary SHALL detect the request and THE AI_Assistant SHALL refuse with an explanation
3. WHEN a patient describes symptoms for interpretation, THE Clinical_Boundary SHALL detect the request and THE AI_Assistant SHALL refuse with an explanation
4. WHEN a patient asks for health advice, THE Clinical_Boundary SHALL detect the request and THE AI_Assistant SHALL refuse with an explanation
5. THE AI_Assistant SHALL only collect non-clinical information such as department selection and reason for visit (administrative purpose)
6. THE Intent_Classifier SHALL categorize all patient requests and flag clinical requests for rejection

### Requirement 6: Conversational AI and Natural Language Processing

**User Story:** As a patient, I want to interact with the system using natural language in my preferred language, so that I can communicate comfortably.

#### Acceptance Criteria

1. THE AI_Assistant SHALL support voice and text input across all channels
2. THE Speech_Service SHALL convert speech to text for voice interactions
3. THE Speech_Service SHALL convert text to speech for voice responses
4. THE Intent_Classifier SHALL determine patient intent from natural language input
5. THE AI_Assistant SHALL support Hindi and configurable regional languages
6. THE Session_Manager SHALL maintain dialogue context across multiple conversation turns
7. WHEN the AI_Assistant cannot understand a request, THE system SHALL ask clarifying questions

### Requirement 7: Receipt and Confirmation Generation

**User Story:** As a patient, I want to receive receipts and confirmations, so that I have proof of my registration and booking.

#### Acceptance Criteria

1. WHEN a registration or booking is completed, THE Receipt_Generator SHALL create a receipt containing patient name, token or appointment ID, department, date/time, and navigation instructions
2. WHERE the channel is QR code/web, THE Receipt_Generator SHALL provide a PDF receipt for download
3. WHERE the channel is SMS, THE system SHALL send a text confirmation with booking details
4. WHERE the channel is hospital kiosk, THE Receipt_Generator SHALL print a physical receipt
5. WHERE the channel is phone call, THE system SHALL send an SMS confirmation after the call
6. THE Receipt_Generator SHALL format receipts consistently across all channels

### Requirement 8: Backend API and Session Management

**User Story:** As a system architect, I want a stateless REST API with session management, so that the system scales efficiently and maintains conversation context.

#### Acceptance Criteria

1. THE system SHALL implement a Python FastAPI backend with stateless REST endpoints
2. THE Session_Manager SHALL store session state in a persistent data store (DynamoDB or RDS PostgreSQL)
3. WHEN a patient initiates a conversation, THE Session_Manager SHALL create a unique session identifier
4. WHEN a patient sends a message, THE API SHALL retrieve session context, process the request, and update session state
5. THE API SHALL implement endpoints for registration, booking, navigation, OTP verification, and receipt generation
6. THE API SHALL validate all input data and return appropriate error responses for invalid requests
7. THE API SHALL implement rate limiting to prevent abuse

### Requirement 9: ERP Integration

**User Story:** As a hospital administrator, I want the system to integrate with our existing ERP, so that patient data flows seamlessly into our hospital management system.

#### Acceptance Criteria

1. WHEN a patient completes registration, THE ERP_Integration SHALL create a patient record in the hospital ERP system
2. WHEN an appointment is booked, THE ERP_Integration SHALL create an appointment record in the hospital ERP system
3. WHEN a queue token is generated, THE ERP_Integration SHALL register the token in the hospital ERP system
4. THE ERP_Integration SHALL implement retry logic for failed ERP calls
5. IF an ERP call fails after retries, THEN THE system SHALL log the error and notify hospital administrators
6. THE ERP_Integration SHALL support configurable ERP endpoints and authentication methods

### Requirement 10: AWS Cloud Architecture

**User Story:** As a system architect, I want the system deployed on AWS with proper cloud services, so that it is scalable, reliable, and secure.

#### Acceptance Criteria

1. THE system SHALL use AWS API Gateway or Application Load Balancer for request routing
2. THE system SHALL use AWS Lambda or EC2 for compute resources
3. THE system SHALL use DynamoDB or RDS PostgreSQL for data persistence
4. THE system SHALL use S3 for storing PDF receipts and static assets
5. THE system SHALL use SNS or SQS for asynchronous tasks such as SMS sending and notifications
6. THE system SHALL use CloudWatch for logging, monitoring, and alerting
7. THE system SHALL use IAM for access control and service permissions
8. THE system SHALL use Secrets Manager for storing credentials and API keys
9. THE system SHALL integrate with SMS and IVR gateway services for phone and SMS channels
10. THE system SHALL support deployment to AWS cloud and optional on-premises or hospital-local infrastructure

### Requirement 11: Data Security and Privacy

**User Story:** As a hospital administrator, I want patient data to be secure and private, so that we comply with data protection regulations.

#### Acceptance Criteria

1. THE system SHALL encrypt all data in transit using TLS 1.2 or higher
2. THE system SHALL encrypt all data at rest using AES-256 or equivalent
3. THE system SHALL store only minimal personally identifiable information (name, mobile number, age)
4. THE system SHALL NOT store medical history, symptoms, diagnoses, or clinical information
5. THE Audit_Log SHALL record all registrations, bookings, and system access with timestamps and user identifiers
6. THE system SHALL implement configurable data retention policies per hospital
7. THE system SHALL support data deletion requests for patient records
8. THE system SHALL implement role-based access control for Hospital_Admin users

### Requirement 12: Scalability and Performance

**User Story:** As a hospital administrator, I want the system to handle high daily footfall, so that it performs reliably during peak hours.

#### Acceptance Criteria

1. THE system SHALL support at least 10,000 concurrent patient sessions
2. THE system SHALL respond to API requests within 2 seconds under normal load
3. THE system SHALL scale compute resources automatically based on demand
4. THE system SHALL implement database connection pooling for efficient resource usage
5. THE system SHALL cache frequently accessed data (hospital departments, services) to reduce database load
6. THE system SHALL implement circuit breakers for external service calls (ERP, SMS gateway)

### Requirement 13: Multi-Hospital Support

**User Story:** As a hospital chain administrator, I want to deploy the system across multiple hospitals, so that we can standardize patient experience across our network.

#### Acceptance Criteria

1. THE system SHALL support Multi_Hospital_Instance configurations with separate data isolation
2. WHEN a patient accesses the system, THE system SHALL identify the hospital from the access channel (QR code, phone number, kiosk)
3. THE system SHALL maintain separate ERP integrations for each hospital
4. THE system SHALL support hospital-specific configurations (departments, services, languages, OTP settings)
5. THE system SHALL provide hospital-specific analytics and reporting
6. THE system SHALL support centralized administration for hospital chains

### Requirement 14: Administration and Analytics

**User Story:** As a hospital administrator, I want to access system analytics and configuration, so that I can monitor performance and adjust settings.

#### Acceptance Criteria

1. THE system SHALL provide a web-based admin interface for Hospital_Admin users
2. THE admin interface SHALL display analytics including daily registrations, bookings, channel usage, and peak hours
3. THE admin interface SHALL allow configuration of hospital departments, services, languages, and OTP settings
4. THE admin interface SHALL display system health metrics (API response times, error rates, resource usage)
5. THE admin interface SHALL provide access to Audit_Log records for compliance and troubleshooting
6. THE admin interface SHALL support exporting analytics data in CSV or PDF format
7. THE admin interface SHALL implement authentication and role-based access control

### Requirement 15: SMS and Phone Integration

**User Story:** As a patient, I want to interact with the system via SMS and phone calls, so that I can access services without internet connectivity.

#### Acceptance Criteria

1. WHEN a patient sends an SMS to the hospital number, THE system SHALL parse the message and process registration or booking requests
2. WHEN a patient calls the hospital number, THE system SHALL answer with an AI voice assistant
3. THE system SHALL integrate with an SMS gateway for sending and receiving text messages
4. THE system SHALL integrate with an IVR gateway for handling voice calls
5. THE Speech_Service SHALL support real-time speech-to-text during phone calls
6. THE Speech_Service SHALL support real-time text-to-speech for voice responses
7. THE system SHALL handle call disconnections gracefully and send SMS summaries of incomplete interactions

### Requirement 16: Kiosk Hardware Integration

**User Story:** As a patient, I want to use a hospital kiosk with voice interaction and receipt printing, so that I can complete registration on-site.

#### Acceptance Criteria

1. THE kiosk interface SHALL run on tablet devices with touchscreen and microphone
2. THE kiosk interface SHALL support voice-first interaction with on-screen visual feedback
3. THE kiosk interface SHALL integrate with a receipt printer for physical receipt generation
4. WHEN a patient completes a transaction at a kiosk, THE Receipt_Generator SHALL send a print command to the connected printer
5. THE kiosk interface SHALL support multiple languages with easy language selection
6. THE kiosk interface SHALL reset to the home screen after 30 seconds of inactivity
7. THE kiosk interface SHALL work offline with cached hospital data and queue when connectivity is restored

### Requirement 17: Error Handling and Resilience

**User Story:** As a system architect, I want the system to handle errors gracefully, so that patients receive helpful feedback and the system remains operational.

#### Acceptance Criteria

1. WHEN an external service (ERP, SMS gateway) is unavailable, THE system SHALL queue requests and retry with exponential backoff
2. WHEN a database operation fails, THE system SHALL log the error and return a user-friendly error message
3. WHEN the Speech_Service fails to recognize speech, THE AI_Assistant SHALL ask the patient to repeat or use text input
4. WHEN the Intent_Classifier cannot determine intent with confidence, THE AI_Assistant SHALL ask clarifying questions
5. THE system SHALL implement health check endpoints for all services
6. THE system SHALL send alerts to Hospital_Admin users when critical errors occur
7. THE system SHALL maintain a 99.5% uptime SLA during business hours

### Requirement 18: Localization and Accessibility

**User Story:** As a patient with limited technical skills or language barriers, I want the system to be accessible and available in my language, so that I can use it comfortably.

#### Acceptance Criteria

1. THE AI_Assistant SHALL support Hindi and at least two configurable regional languages
2. THE system SHALL detect patient language preference from initial input or allow manual selection
3. THE system SHALL provide voice interaction for patients who cannot read or type
4. THE user interface SHALL use large fonts and high-contrast colors for readability
5. THE AI_Assistant SHALL use simple, clear language and avoid technical jargon
6. THE system SHALL support slow speech recognition for elderly patients
7. THE kiosk interface SHALL provide audio instructions for visually impaired patients
