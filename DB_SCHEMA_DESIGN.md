# WorkFlow HRMS - Database Schema Design

## Core Entities

### 1. User & Role (UserFeature)
- **Employee** (inherits from UserBase)
  - EmployeeId (Primary Key)
  - FirstName, LastName
  - Email, Phone
  - Designation
  - Department
  - DateOfJoining
  - ManagerId (Self-reference)
  - ReportingManagerId (Self-reference)
  - Role (Employee, Manager, HR, Admin)
  - Status (Active, Inactive, Onboarding, Exit)
  - Country (IN, US)
  - OnboardingStatus (Pre-joining, Day-1, Week-1, Month-1, Completed)

### 2. Attendance (AttendanceFeature)
- **AttendanceRecord**
  - Id
  - EmployeeId
  - Date
  - ClockInTime
  - ClockOutTime
  - ClockInLocation (Lat, Long)
  - ClockOutLocation (Lat, Long)
  - ClockInIP
  - ClockOutIP
  - ClockInSelfieUrl
  - Status (Present, Absent, Late, HalfDay, OnLeave)
  - TotalHours
  - OvertimeHours

### 3. Leave Management (LeaveFeature)
- **LeaveRequest**
  - Id
  - EmployeeId
  - LeaveType (Casual, Sick, Personal, Maternity, Paternity, LWP, CompOff)
  - StartDate, EndDate
  - Reason
  - Status (Pending, Approved, Rejected, Cancelled)
  - ApprovedByManagerId
  - ApprovedByHRId
- **LeaveBalance**
  - Id
  - EmployeeId
  - LeaveType
  - TotalEntitled
  - Used
  - Pending
  - CarriedForward

### 4. Documents (DocumentFeature)
- **EmployeeDocument**
  - Id
  - EmployeeId
  - Category (Identity, Employment, WorkAuth, Tax, Education, Other)
  - DocumentName
  - FileUrl
  - ExpiryDate
  - Status (Missing, Uploaded, Verified, Rejected)
  - RejectionReason

### 5. Payroll (PayrollFeature)
- **PayrollRecord**
  - Id
  - EmployeeId
  - Month, Year
  - GrossPay, NetPay
  - TotalEarnings, TotalDeductions
  - Status (Draft, Processing, Approved, Paid)
- **PayrollComponent**
  - Id
  - PayrollRecordId
  - Name (Basic, HRA, PF, etc.)
  - Amount
  - Type (Earning, Deduction, EmployerContribution)

### 6. Expenses (ExpenseFeature)
- **ExpenseClaim**
  - Id
  - EmployeeId
  - Category (Travel, Food, Accommodation, etc.)
  - Amount
  - Date
  - Description
  - ReceiptUrl
  - Status (Draft, Submitted, Pending, Approved, Rejected, Paid)
  - Mileage (Distance, VehicleType, Rate)

### 7. Performance (PerformanceFeature)
- **Goal**
  - Id
  - EmployeeId
  - Title, Description
  - Category (Individual, Team, Dept, Org)
  - Weight
  - TargetValue, CurrentValue
  - Status (NotStarted, InProgress, OnTrack, AtRisk, Completed, Cancelled)
- **PerformanceReview**
  - Id
  - EmployeeId
  - ReviewerId
  - Period (Quarterly, Annual, etc.)
  - Rating
  - Feedback
  - Status (Draft, Submitted, Completed)

### 8. Contributions (ContributionFeature)
- **ValueContribution**
  - Id
  - EmployeeId
  - Title, Description
  - Category (Innovation, ProcessImprovement, etc.)
  - Points
  - ImpactLevel
  - EvidenceUrl
  - Status (Proposal, InProgress, UnderReview, Completed)
- **ContributionItem** (Catalog)
  - Id
  - Title, Description
  - PointsRange

### 9. Training (TrainingFeature)
- **TrainingModule**
  - Id
  - Title, Description
  - Category
  - ContentUrl
  - IsMandatory
- **EmployeeTraining**
  - Id
  - EmployeeId
  - TrainingModuleId
  - Progress
  - Status (NotStarted, InProgress, Completed)
  - CompletionDate
  - CertificateUrl

### 10. Recruitment (RecruitmentFeature)
- **JobPosting**
  - Id
  - Title, Description
  - Department, Location
  - Requirements, Responsibilities
  - Status (Open, Closed)
- **Candidate**
  - Id
  - JobPostingId
  - Name, Email, Phone
  - ResumeUrl
  - Status (New, Screening, Shortlisted, Interviewing, Hired, Rejected)

### 11. Recognition (RecognitionFeature)
- **Recognition**
  - Id
  - FromEmployeeId
  - ToEmployeeId
  - Category
  - Message
  - Visibility (Public, Private)

### 12. Announcements (AnnouncementFeature)
- **Announcement**
  - Id
  - Title, Content
  - Category
  - Priority
  - Scope (Global, Dept, Location)
  - CreatedByUserId

### 13. Onboarding (OnboardingFeature)
- **OnboardingTask**
  - Id
  - EmployeeId
  - Phase (PreJoining, Day1, Week1, Month1)
  - Title, Description
  - IsCompleted
- **WelcomeMessage**
  - Id
  - EmployeeId
  - FromRole (CEO, Manager, Buddy, HR)
  - Message, VideoUrl
