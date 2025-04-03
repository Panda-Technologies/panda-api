# RAMvisor Backend API Documentation

## Overview

This document provides detailed information about the backend API for the RAMvisor application - a university degree planning and academic management system. The API is built with GraphQL, TypeScript, Prisma ORM, and Express.

## Tech Stack

- **Node.js/Express**: Web server
- **TypeScript**: Language
- **GraphQL/Apollo Server**: API layer
- **Prisma**: ORM for PostgreSQL
- **Redis**: Session management
- **PostgreSQL**: Database

## Authentication

The API uses session-based authentication with cookies. Session data is stored in Redis.

### Authentication Resolvers

#### Login

```typescript
mutation Login($input: LoginInput!) {
 login(input: $input)
}
```

- **Input**: Email and password
- **Returns**: User ID (String)
- **Description**: Authenticates user and creates a session

#### Logout

```typescript
mutation Logout {
 logout
}
```

- **Returns**: Boolean
- **Description**: Destroys the current session

#### Register

```typescript
mutation Register($input: RegisterInput!) {
 register(input: $input)
}
```

- **Input**: Email and password
- **Returns**: User ID (String)
- **Description**: Creates a new user account

## Data Models

### User

Represents a university student with academic information.

Fields:
- `id`: String (UUID)
- `email`: String (unique)
- `password`: String (hashed)
- `university`: String (optional)
- `yearInUniversity`: Int (optional)
- `isPremium`: Boolean
- `graduationSemesterName`: String (optional)
- `gpa`: Float (optional)
- `attendancePercentage`: Float (optional)
- `assignmentCompletionPercentage`: Float (optional)
- `takenClassIds`: [Int]
- `questionnaireCompleted`: Boolean

Related entities:
- `tasks`: [Task]
- `classSchedule`: [ClassSchedule]
- `degreePlanners`: [DegreePlanner]
- `degrees`: [Degree]
- `classes`: [ClassSection]
- `completedTasks`: [CompletedTasks]

### Class

Represents an academic course.

Fields:
- `id`: Int
- `classCode`: String
- `credits`: Int
- `category`: String
- `description`: String
- `courseType`: String
- `title`: String
- `color`: String
- `coreDegreeId`: [Int]
- `electiveDegreeId`: [Int]

Related entities:
- `sections`: [ClassSection]
- `semesterEntries`: [SemesterEntry]
- `scheduleEntries`: [ClassScheduleEntry]

### ClassSection

Represents a specific section of a class with timing information.

Fields:
- `id`: Int
- `classId`: Int
- `section`: Int
- `professor`: String
- `rateMyProfessorRating`: Float
- `dayOfWeek`: String
- `startTime`: String
- `endTime`: String

### Task

Represents an academic task or assignment.

Fields:
- `id`: Int
- `userId`: String
- `dueDate`: String
- `stageId`: Int
- `classCode`: String (optional)
- `description`: String (optional)
- `source`: String ('canvas' or 'app')
- `title`: String

### CompletedTasks

Tracks completed tasks to avoid showing them in the active task list.

Fields:
- `id`: Int
- `userId`: String
- `classCode`: String
- `title`: String
- `completedAt`: DateTime
- `source`: String

### ClassSchedule

Represents a student's class schedule for a semester.

Fields:
- `id`: Int
- `title`: String (optional)
- `userId`: String
- `semesterId`: String
- `isCurrent`: Boolean (optional)

Related entities:
- \`entries\`: [ClassScheduleEntry]

### ClassScheduleEntry

Maps classes to a schedule.

Fields:
- `id`: Int
- `classScheduleId`: Int
- `classId`: Int
- `sectionId`: Int

### Degree

Represents a university degree program.

Fields:
- `id`: Int
- `name`: String
- `type`: String
- `coreCategories`: [String]
- `electiveCategories`: [String]
- `gatewayCategories`: [String]
- `numberOfCores`: Float
- `numberOfElectives`: Float

Related entities:
- `degreePlanners`: [DegreePlanner]

### DegreePlanner

A tool for planning semester-by-semester degree completion.

Fields:
- `id`: Int
- `userId`: String
- `title`: String
- `degreeId`: Int

Related entities:
- `semester`: [Semester]

### Semester

Represents an academic semester within a degree plan.

Fields:
- `id`: Int
- `degreeId`: Int
- `name`: String
- `credits`: Int
- `plannerId`: Int

Related entities:
- `entries`: [SemesterEntry]

### SemesterEntry

Maps classes to a semester.

Fields:
- `id`: Int
- `semesterId`: Int
- `classId`: Int

### Requirement

Defines a specific degree requirement.

Fields:
- `id`: Int
- `degreeId`: Int
- `reqType`: String
- `category`: String
- `classIds`: [Int]

## API Endpoints

### User Management

#### Get User

```typescript
query GetUser {
 getUser {
   id
   email
   university
   isPremium
   yearInUniversity
   graduationSemesterName
   gpa
   attendancePercentage
   assignmentCompletionPercentage
   takenClassIds
   degrees {
     id
     name
     type
     coreCategories
     gatewayCategories
     electiveCategories
     numberOfCores
     numberOfElectives
   }
 }
}
```

#### Current User (Me)

```typescript
query Me {
 me {
   id
   email
   university
   isPremium
   yearInUniversity
   graduationSemesterName
   gpa
   attendancePercentage
   assignmentCompletionPercentage
   takenClassIds
   degrees {
     id
     name
   }
 }
}
```

#### Update User Profile

```typescript
mutation UpdateUserProfile($university: String!, $yearInUniversity: Int!, $degreeIds: [Int]!) {
 updateUserProfile(university: $university, yearInUniversity: $yearInUniversity, degreeIds: $degreeIds) {
   id
   university
   yearInUniversity
   degrees {
     id
     name
   }
 }
}
```

#### Update User Academic Info

```typescript
mutation UpdateUserAcademicInfo($id: String!, $gpa: Float, $attendancePercentage: Float, $assignmentCompletionPercentage: Float, $takenClassIds: [Int]) {
 updateUserAcademicInfo(id: $id, gpa: $gpa, attendancePercentage: $attendancePercentage, assignmentCompletionPercentage: $assignmentCompletionPercentage, takenClassIds: $takenClassIds) {
   id
   gpa
   attendancePercentage
   assignmentCompletionPercentage
   takenClassIds
 }
}
```

#### Set Graduation Semester

```typescript
mutation SetUserGraduationSemester($input: graduationSemesterInput!) {
 setUserGraduationSemester(input: $input) {
   id
   graduationSemesterName
 }
}
```

#### Update Premium Status

```typescript
mutation UpdatePremiumStatus($id: String!, $isPremium: Boolean!) {
 updatePremiumStatus(id: $id, isPremium: $isPremium) {
   id
   isPremium
 }
}
```

#### Mark Questionnaire Completed

```typescript
mutation MarkQuestionnaireCompleted($isQuestionnaireCompleted: Boolean) {
 updateUserProfile(questionnaireCompleted: $isQuestionnaireCompleted) {
   id
   isQuestionnaireCompleted
 }
}
```

### Class Management

#### Get Classes

```typescript
query GetClasses {
 getClasses {
   id
   classCode
   credits
   courseType
   title
   description
   category
   sections {
     id
     section
     classId
     dayOfWeek
     startTime
     endTime
     professor
     rateMyProfessorRating
   }
   color
   coreDegreeId
   electiveDegreeId
 }
}
```

#### Get Single Class

```typescript
query GetClass($id: Int!) {
 getClass(id: $id) {
   id
   classCode
   credits
   courseType
   title
   description
   category
   sections {
     id
     dayOfWeek
     startTime
     endTime
     professor
     rateMyProfessorRating
   }
   color
   coreDegreeId
   electiveDegreeId
 }
}
```

#### Create Class

```typescript
mutation CreateClass($input: CreateClassInput!) {
 createClass(input: $input) {
   id
   classCode
   credits
   courseType
   title
   description
   category
   color
   coreDegreeId
   electiveDegreeId
 }
}
```

#### Update Class

```typescript
mutation UpdateClass($input: UpdateClassInput!) {
 updateClass(input: $input) {
   id
   classCode
   credits
   courseType
   title
   description
   category
   color
   coreDegreeId
   electiveDegreeId
 }
}
```

#### Delete Class

```typescript
mutation DeleteClass($input: deleteClassInput!) {
 deleteClass(input: $input) {
   id
 }
}
```

### Class Schedule Management

#### Get Class Schedules

```typescript
query GetClassSchedules {
 getClassSchedules {
   id
   title
   semesterId
   entries {
     id
     classScheduleId
     classId
     sectionId
     class {
       id
       classCode
       credits
       courseType
       title
       description
       category
       sections {
         id
         section
         classId
         dayOfWeek
         startTime
         endTime
         professor
         rateMyProfessorRating
       }
       color
     }
   }
 }
}
```

#### Create Class Schedule

```typescript
mutation CreateClassSchedule($input: createClassScheduleInput!) {
 createClassSchedule(input: $input) {
   id
   title
   semesterId
   entries {
     id
     classScheduleId
     classId
     sectionId
     class {
       id
       classCode
       credits
       courseType
       title
       description
       category
       sections {
         id
         section
         classId
         dayOfWeek
         startTime
         endTime
         professor
         rateMyProfessorRating
       }
       color
     }
   }
 }
}
```

#### Add Class to Schedule

```typescript
mutation AddClassToClassSchedule($input: addClassToScheduleInput!) {
 addClassToClassSchedule(input: $input) {
   id
   classScheduleId
   sectionId
   class {
     id
     classCode
     credits
     courseType
     title
     description
     category
     sections {
       id
       section
       classId
       dayOfWeek
       startTime
       endTime
       professor
       rateMyProfessorRating
     }
     color
   }
 }
}
```

#### Remove Class from Schedule

```typescript
mutation RemoveClassFromClassSchedule($input: removeClassFromScheduleInput!) {
 removeClassFromClassSchedule(input: $input) {
   id
 }
}
```

#### Reset Class Schedule

```typescript
mutation ResetClassSchedule($input: resetClassScheduleInput!) {
 resetClassSchedule(input: $input) {
   id
 }
}
```

### Degree Management

#### Get All Degrees

```typescript
query GetAllDegrees {
 getAllDegrees {
   id
   name
   coreCategories
   gatewayCategories
   electiveCategories
   numberOfCores
   numberOfElectives
 }
}
```

#### Get User's Degree

```typescript
query GetDegree {
 getDegree {
   id
   name
   coreCategories
   gatewayCategories
   electiveCategories
   numberOfCores
   numberOfElectives
 }
}
```

#### Create Degree

```typescript
mutation CreateDegree($input: createDegreeInput!) {
 createDegree(input: $input) {
   id
   name
   coreCategories
   gatewayCategories
   electiveCategories
   numberOfCores
   numberOfElectives
 }
}
```

### Degree Planner Management

#### Get Degree Planners

```typescript
query GetDegreePlanners {
 getDegreePlanners {
   id
   title
   userId
   degreeId
   semester {
     id
     name
     credits
     entries {
       id
       classId
       class {
         id
         classCode
         title
         credits
       }
     }
   }
 }
}
```

#### Create Degree Planner

```typescript
mutation CreateDegreePlanner($input: createDegreePlannerInput!) {
 createDegreePlanner(input: $input) {
   id
   title
   userId
   degreeId
   semester {
     id
     name
     credits
     degreeId
     plannerId
   }
 }
}
```

#### Reset Degree Planner

```typescript
mutation ResetDegreePlanner($input: resetDegreePlannerInput!) {
 resetDegreePlanner(input: $input) {
   id
   title
   userId
   degreeId
 }
}
```

### Semester Management

#### Get Semesters

```typescript
query GetSemesters($plannerId: Int!) {
 getSemesters(plannerId: $plannerId) {
   id
   name
   credits
   degreeId
   plannerId
   entries {
     id
     classId
     class {
       id
       classCode
       credits
       title
     }
   }
 }
}
```

#### Create Semester

```typescript
mutation CreateSemester($input: createSemesterInput!) {
 createSemester(input: $input) {
   id
   name
   credits
   degreeId
   plannerId
   entries {
     id
     classId
     class {
       id
       classCode
       title
       credits
     }
   }
 }
}
```

#### Update Semester

```typescript
mutation UpdateSemester($input: updateSemesterInput!) {
 updateSemester(input: $input) {
   id
   name
   credits
   entries {
     id
     classId
     class {
       id
       classCode
       title
       credits
     }
   }
 }
}
```

#### Add Class to Semester

```typescript
mutation AddClassToSemester($input: addClassToSemesterInput!) {
 addClassToSemester(input: $input) {
   id
   semesterId
   classId
   class {
     id
     classCode
     title
     credits
   }
 }
}
```

#### Remove Class from Semester

```typescript
mutation RemoveClassFromSemester($input: removeClassFromSemesterInput!) {
 removeClassFromSemester(input: $input) {
   id
   class {
     id
     classCode
     title
     credits
   }
 }
}
```

### Task Management

#### Get Tasks

```typescript
query GetTasks {
 getTasks {
   id
   userId
   dueDate
   stageId
   classCode
   description
   title
 }
}
```

#### Create Task

```typescript
mutation CreateTask($input: createTaskInput!) {
 createTask(input: $input) {
   id
   userId
   dueDate
   stageId
   classCode
   description
   title
 }
}
```

#### Update Task

```typescript
mutation UpdateTask($input: updateTaskInput!) {
 updateTask(input: $input) {
   id
   title
   dueDate
   stageId
   classCode
   description
 }
}
```

#### Delete Task

```typescript
mutation DeleteTask($input: deleteTaskInput!) {
 deleteTask(input: $input) {
   id
 }
}
```

### Requirements Management

#### Get Requirements

```typescript
query GetRequirements($degreeId: Int!) {
 getRequirements(degreeId: $degreeId) {
   id
   category
   reqType
   classIds
   degreeId
 }
}
```

## Utility Features

### Class Import from Canvas

```typescript
mutation ImportCanvasClasses($input: importCanvasClassesInput!) {
 importCanvasClasses(input: $input) {
   id
   classCode
   title
 }
}
```

### Task Import from Canvas

```typescript
mutation ImportCanvasTasks($input: importCanvasTasksInput!) {
 importCanvasTasks(input: $input) {
   id
   title
   dueDate
 }
}
```

## Error Handling

The API returns GraphQL errors with descriptive messages. Common error patterns:

- Authentication errors (401)
- Not found errors (404)
- Validation errors (400)
- Server errors (500)

## Environment Variables

- `DATABASE_URL`: PostgreSQL connection string
- `SESSION_SECRET`: Secret for session cookies
- `REDIS_HOST`: Redis host (default: localhost)
- `REDIS_PORT`: Redis port (default: 6379)
- `PORT`: Server port (default: 5001)

## Deployment

The application is designed to be deployed as a Node.js service with connections to PostgreSQL and Redis databases.

## Development

1. Clone the repository
2. Install dependencies: `npm install`
3. Configure environment variables
4. Run migrations: `npx prisma migrate dev`
5. Start development server: `npm run dev`

## Security Considerations

- Passwords are hashed using Argon2
- Sessions use HttpOnly cookies
- Authentication is required for most operations
- Input validation is performed using GraphQL schema
