# Church CRM – Data Model

This document outlines the conceptual data model for the Church CRM system, including entity definitions, relationships, and attributes.

---

## 🧍 1. Member

Stores data about individuals in the church.

| Field             | Type        | Description                            |
|------------------|-------------|----------------------------------------|
| member_id        | UUID / INT  | Primary key                            |
| first_name       | String      |                                        |
| last_name        | String      |                                        |
| email            | String      | Unique                                 |
| phone            | String      |                                        |
| address          | String      |                                        |
| date_of_birth    | Date        |                                        |
| gender           | Enum        | Male, Female, Other                    |
| marital_status   | Enum        | Single, Married, etc.                  |
| join_date        | Date        |                                        |
| membership_status| Enum        | Visitor, Member, Inactive              |
| notes            | Text        | Confidential pastoral notes            |
| family_id        | FK          | References `Family`                    |

---

## 👨‍👩‍👧‍👦 2. Family

Represents a household unit.

| Field       | Type       | Description              |
|-------------|------------|--------------------------|
| family_id   | UUID / INT | Primary key              |
| family_name | String     | e.g., "Smith Family"     |
| address     | String     | Shared household address |

---

## 🤝 3. Group

Small group, ministry team, or class.

| Field        | Type       | Description                             |
|--------------|------------|-----------------------------------------|
| group_id     | UUID / INT | Primary key                             |
| name         | String     | Name of the group                       |
| type         | Enum       | Small Group, Ministry, Class            |
| description  | Text       |                                         |
| meeting_day  | String     | e.g., "Wednesdays"                      |
| leader_id    | FK         | References `Member` (group leader)      |

---

## 📋 4. GroupMembership

Join table for members in groups.

| Field         | Type       | Description                        |
|---------------|------------|------------------------------------|
| membership_id | UUID / INT | Primary key                        |
| member_id     | FK         | References `Member`                |
| group_id      | FK         | References `Group`                 |
| joined_on     | Date       | Date member joined the group       |
| role          | String     | Leader, Member, Host, etc.         |

---

## 📆 5. Event

Represents services, outreach, meetings.

| Field       | Type       | Description                          |
|-------------|------------|--------------------------------------|
| event_id    | UUID / INT | Primary key                          |
| name        | String     | Event name                           |
| description | Text       | Event details                        |
| date_time   | DateTime   | Date and time                        |
| location    | String     | Physical or virtual location         |
| group_id    | FK         | Optional – references `Group`        |
| capacity    | Integer    | Optional max attendees               |

---

## 🧾 6. Attendance

Tracks which members attended which events.

| Field          | Type       | Description                          |
|----------------|------------|--------------------------------------|
| attendance_id  | UUID / INT | Primary key                          |
| member_id      | FK         | References `Member`                  |
| event_id       | FK         | References `Event`                   |
| check_in_time  | DateTime   | Optional                             |
| notes          | Text       | Optional                             |

---

## 💰 7. Donation

Records financial giving activity.

| Field        | Type       | Description                                |
|--------------|------------|--------------------------------------------|
| donation_id  | UUID / INT | Primary key                                |
| member_id    | FK         | Nullable (anonymous gifts) – `Member`      |
| amount       | Decimal    | Amount given                               |
| date_given   | Date       |                                            |
| campaign_id  | FK         | Optional – references `Campaign`           |
| method       | Enum       | Cash, Card, Bank Transfer, Online          |
| note         | Text       | e.g., "In memory of..."                    |

---

## 📢 8. Campaign

Fundraising campaigns (e.g., Building Fund).

| Field        | Type       | Description                      |
|--------------|------------|----------------------------------|
| campaign_id  | UUID / INT | Primary key                      |
| name         | String     | Campaign title                   |
| goal_amount  | Decimal    | Fundraising goal                 |
| start_date   | Date       |                                  |
| end_date     | Date       |                                  |
| description  | Text       | Campaign purpose/details         |

---

## ✉️ 9. CommunicationLog

Tracks emails, SMS, and letters sent to members.

| Field      | Type       | Description                          |
|------------|------------|--------------------------------------|
| comm_id    | UUID / INT | Primary key                          |
| member_id  | FK         | References `Member`                  |
| type       | Enum       | Email, SMS, Push, Mail               |
| subject    | String     |                                     |
| body       | Text       |                                     |
| sent_at    | DateTime   |                                     |
| status     | Enum       | Sent, Failed, Scheduled              |

---

## 🙋 10. VolunteerRole

Describes roles members play in volunteer efforts.

| Field         | Type       | Description                        |
|---------------|------------|------------------------------------|
| volunteer_id  | UUID / INT | Primary key                        |
| member_id     | FK         | References `Member`                |
| role_name     | String     | e.g., Usher, Teacher, Greeter      |
| ministry_area | String     | e.g., Kids, Hospitality, Worship   |
| start_date    | Date       |                                    |
| end_date      | Date       | Optional                           |

---

## 🧭 Optional Tables

### AdminUser
For staff/admin access control.

| Field       | Type       | Description            |
|-------------|------------|------------------------|
| admin_id    | UUID / INT | Primary key            |
| email       | String     | Login email            |
| password    | String     | Hashed                 |
| role        | Enum       | Admin, Staff, Volunteer|

### AuditLog
Tracks internal changes for security.

| Field       | Type       | Description                      |
|-------------|------------|----------------------------------|
| log_id      | UUID / INT | Primary key                      |
| actor_id    | FK         | Admin who performed action       |
| action      | String     | Create, Update, Delete           |
| entity_type | String     | Member, Donation, etc.           |
| entity_id   | UUID / INT | ID of the modified entity        |
| timestamp   | DateTime   | When the action occurred         |

---

## 🔗 Entity Relationships

- `Member` belongs to `Family`
- `Member` has many `Donations`, `Attendances`, `VolunteerRoles`, and `Communications`
- `Member` belongs to many `Groups` through `GroupMembership`
- `Group` has many `Events`
- `Event` has many `Attendances`
- `Donation` can optionally belong to a `Campaign`

Member ────────┐
├──< GroupMembership >── Group
└──< Attendance >── Event
Member ───< Donation >── Campaign
Member ───< CommunicationLog
Member ───< VolunteerRole
Member ─── belongs to ─── Family
Group ───< Event

yaml
Copy
Edit

---

## 📌 Notes

- Use UUIDs for scalability and uniqueness.
- Implement soft-deletes (e.g., `is_active` flags).
- Ensure role-based access control for data privacy.
- Add indexes to foreign keys and commonly queried fields.
- Store files (PDFs, images) in a blob/object store and link in the DB.
