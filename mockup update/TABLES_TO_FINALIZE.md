# HireScope table catalogue — pending finalisation

**Status:** Complete replacement table catalogue. The generated `pente_db_diagram.dbml` is structurally validated; remaining product-sign-off items are recorded in `SCHEMA_VALIDATION.md`.

## Confirmed roles

The prototype and replacement schema use only these roles:

1. Organization Owner
2. Organization Admin
3. HR
4. Hiring Manager
5. Interviewer
6. Applicant

Platform Admin and Requisition Creator are not roles in this replacement model.

Paste the Google Sheet contents below. A Markdown table, CSV, or a simple list is fine. For each table, include the table name, purpose, primary key, key relationships, lifecycle/status fields, tenant scope, and any screen or workflow that creates, reads, or updates it.

## Paste the final table list here
1	organization_onboarding_requests																									
	Column	Purpose	PK	FK	Data Type																					
	onboarding_request_id	Identifies each new-organization request	PK	—	UUID																					
	requester_keycloak_user_id	Identifies the proposed Organization Admin	—	FK → users.keycloak_user_id	UUID																					
	organization_name	Identifies the proposed workspace	—	—	VARCHAR(150)																					
	document_url	Allows Platform Admin to verify the organization	—	—	TEXT																					
	comment	Provides onboarding context	—	—	TEXT																					
	request_status	PENDING, APPROVED, REJECTED, or PROVISIONING_FAILED	—	—	VARCHAR(30)																					
	rejection_reason	Required when request is rejected	—	—	TEXT																					
	approved_organization_id	Links successful request to workspace	—	FK → organizations.organization_id	UUID																					
	provisioning_status	Tracks NOT_STARTED, PROCESSING, COMPLETED, or FAILED	—	—	VARCHAR(30)																					
	provisioning_error	Supports safe provisioning retry	—	—	TEXT																					
	provisioning_attempts	Tracks retry count	—	—	INTEGER																					
	last_provisioning_attempt_at	Audit and retry tracking	—	—	TIMESTAMPTZ																					
	created_at	Lifecycle tracking	—	—	TIMESTAMPTZ																					
	updated_at	Lifecycle tracking	—	—	TIMESTAMPTZ																					
																										
2	users																									
	Column	Purpose	PK	FK	Data Type																					
	keycloak_user_id	Unique identity of the user from Keycloak	PK	—	UUID																					
	canonical_person_id	Links realm-local user identity to the verified cross-realm person	—	FK → canonical_people.canonical_person_id	UUID																					
	email	User email	—	—	VARCHAR(255)																					
	keycloak_realm	Realm containing the Keycloak account	—	—	VARCHAR(50)																					
	first_name	First name	—	—	VARCHAR(100)																					
	last_name	Last name	—	—	VARCHAR(100)																					
	phone	Phone number	—	—	VARCHAR(20)																					
	is_assigned	Distinguishes unassigned onboarding users from organization-linked users	—	—	BOOLEAN																					
	status	Platform account status	—	—	VARCHAR(20)																					
	created_at	Registration time	—	—	TIMESTAMPTZ																					
	updated_at	Last update time	—	—	TIMESTAMPTZ																					
																										
3	role_verification_requests																									
	Column	Purpose	PK	FK	Data Type																					
	verification_id	Identifies each staff-role verification request	PK	—	UUID																					
	keycloak_user_id	Identifies the registered user	—	FK → users.keycloak_user_id	UUID																					
	organization_id	Identifies the target organisation/workspace	—	FK → organizations.organization_id	UUID																					
	requested_role	Identifies the intended organisation role	—	—	VARCHAR(30)																					
	document_url	Allows Portal Admin to verify the requested role	—	—	TEXT																					
	membership_id	Links request to organization membership	—	FK → organization_memberships.membership_id	UUID																					
	job_opening_id	Required for Interviewer and later job-specific assignments; NULL for HR Reviewer and initial Hiring Manager request	—	FK → job_openings.job_opening_id	UUID																					
	rejection_reason	Explains rejected access request	—	—	TEXT																					
	supersedes_verification_id	Links corrected resubmission while preserving history	—	FK → role_verification_requests.verification_id	UUID																					
	comment	Provides additional context for verification	—	—	TEXT																					
	verification_status	Tracks whether request is pending, approved, or rejected	—	—	VARCHAR(20)																					
	reviewed_by_membership_id	Reviewer who decided the requisition-creator/Hiring Manager access request	—	FK → organization_memberships.membership_id	UUID																					
	reviewed_at	Role-request decision timestamp	—	—	TIMESTAMPTZ																					
																										
																										
4	organizations																									
	Column	Definition	Purpose	PK	FK	Data Type																				
	organization_id	Unique organisation identifier	Identifies each workspace	PK	—	UUID																				
	organization_name	Name of the organisation	Identifies the workspace	—	—	VARCHAR(150)																				
	keycloak_realm	PostgreSQL should know which Keycloak realm represents that organization.	Workspace identification	—	—	VARCHAR(50)																				
	status	Current organisation status	Controls whether organisation is active	—	—	VARCHAR(20)																				
	created_at	Organisation creation timestamp	Lifecycle/audit tracking	—	—	TIMESTAMPTZ																				
	updated_at	Last organisation update timestamp	Lifecycle/audit tracking	—	—	TIMESTAMPTZ																				
																										
5	organization_memberships																									
	Column	Definition	Purpose	PK	FK	Data Type																				
	membership_id	Unique membership identifier	Identifies a user's membership in an organisation	PK	—	UUID																				
	organization_id	Organisation to which the user belongs	Links member to workspace	—	FK → organizations.organization_id	UUID																				
	keycloak_user_id	User who belongs to the organisation	Links membership to the registered user	—	FK → users.keycloak_user_id	UUID																				
	membership_status	Current membership state	Controls whether membership is active	—	—	VARCHAR(20)																				
	created_at	Membership creation timestamp	Tracks when user became a member	—	—	TIMESTAMPTZ																				
	updated_at	Last membership update timestamp	Tracks membership changes	—	—	TIMESTAMPTZ																				
																										
6	organization_member_roles						Why not put keycloak_user_id directly in organization_member_roles? coz Now the same user has a role in an organisation where they don't have a corresponding membership.																			
	Column	Purpose	PK	FK	Data Type	Data Type																				
	member_role_id	Role-assignment identifier	PK	—	UUID	UUID																				
	membership_id	Staff member receiving the role	—	FK → organization_memberships.membership_id	UUID	UUID																				
	role	ORGANIZATION_ADMIN, HR, HIRING_MANAGER, or INTERVIEWER	—	—	VARCHAR(30)	VARCHAR(30)																				
	job_opening_id	Required for opening-scoped Interviewer; nullable for organisation-scoped roles and pre-opening Hiring Manager access	—	FK → job_openings.job_opening_id	UUID	UUID																				
	source_nomination_id	Approved interviewer nomination that created this role; otherwise NULL	—	FK → opening_stage_interviewer_nominations.nomination_id	UUID																					
	assignment_status	ACTIVE, REVOKED, EXPIRED, or REPLACED	—	—	VARCHAR(30)																					
	revoked_at	Future-access revocation time	—	—	TIMESTAMPTZ																					
	revocation_reason	Reason for revocation/replacement	—	—	TEXT																					
	created_at	Assignment creation time	—	—	TIMESTAMPTZ	TIMESTAMPTZ																				
																										
7	job_openings																									
	Column	Purpose	PK	FK	Data Type																					
	job_opening_id	Opening identifier	PK	—	UUID																					
	organization_id	Owning organisation	—	FK → organizations.organization_id	UUID																					
	created_by_membership_id	Hiring Manager who created the opening	—	FK → organization_memberships.membership_id	UUID																					
	requisition_id	Approved source requisition	—	FK → job_requisitions.requisition_id	UUID																					
	title	Opening title	—	—	VARCHAR(150)																					
	description	Opening description	—	—	TEXT																					
	target_headcount	Positions to fill, not candidate count	—	—	INTEGER																					
	work_location	Confirmed location	—	—	VARCHAR(150)																					
	target_start_date	Target joining date	—	—	DATE																					
	active_opening_version_id	Published version for new applicants	—	FK → opening_versions.opening_version_id	UUID																					
	status	DRAFT, PENDING_OPENING_APPROVAL, RETURNED, APPROVED_TO_PUBLISH, PUBLISHED, SUSPENDED, or CLOSED	—	—	VARCHAR(30)																					
	published_at	First publication time	—	—	TIMESTAMPTZ																					
	suspended_at	Suspension time	—	—	TIMESTAMPTZ																					
	closed_at	Closure time	—	—	TIMESTAMPTZ																					
	created_at	Creation time	—	—	TIMESTAMPTZ																					
	updated_at	Last update time	—	—	TIMESTAMPTZ																					
																										
8	job_applications																									
	Column	Definition	Purpose	PK	FK	Data Type																				
	application_id	Unique application	Identifies candidate's application	PK	—	UUID																				
	job_opening_id	Applied opening	Identifies position	—	FK → job_openings.job_opening_id	UUID																				
	keycloak_user_id	Applicant identity	Identifies candidate	—	FK → users.keycloak_user_id	UUID																				
	current_stage_id	Applicant's current configured stage	Supports current-stage display and progression	—	FK → job_opening_stages.stage_id	UUID																				
	opening_version_id	Published opening version used by the applicant	—	FK → opening_versions.opening_version_id	UUID																					
	canonical_person_id	Verified applicant identity; applying never creates an organization membership	—	FK → canonical_people.canonical_person_id	UUID																					
	replaces_application_id	Linked replacement application after an eligible six-month retest/reapplication	—	FK → job_applications.application_id	UUID																					
	interest_registered_at	Initial Apply/interest-registration timestamp	—	—	TIMESTAMPTZ																					
	status_updated_at	Last applicant-lifecycle status timestamp	—	—	TIMESTAMPTZ																					
	retest_eligible_at	Earliest permitted replacement-application date after a valid incomplete/withdrawal outcome	—	—	TIMESTAMPTZ																					
	status	INTEREST_REGISTERED, APPLICATION_DETAILS_REQUIRED, RESUME_SUBMITTED, AI_PRE_SCREENING, PRE_SCREEN_ANALYSIS_READY, UNDER_HR_PRE_SCREEN_REVIEW, QUALIFIED_FOR_ASSESSMENT, NOT_CURRENTLY_QUALIFIED, PRE_SCREEN_PENDING, ASSESSMENT_INVITED, READINESS_PENDING, READINESS_CONFIRMED, ASSESSMENT_STARTED, ASSESSMENT_SUBMITTED, ASSESSMENT_GRIEVANCE_SUBMITTED, AI_PROCESSING, ANALYSIS_READY, AI_PROCESSING_FAILED, LIVE_INTERVIEW_RECOMMENDED, LIVE_INTERVIEW_HOLD, NOT_PROCEEDING, INTERVIEW_ASSIGNED, INTERVIEW_COMPLETED, ADVANCED, HOLD, REJECTED, SELECTED, ONBOARDING_PENDING, ONBOARDING_COMPLETED, POSITION_FILLED, or APPLICATION_SUSPENDED_NO_RESPONSE	—	—	VARCHAR(50)																					
	distance_traveled_context	Blind-review context, revealed only after score-lock	Blind-review context, revealed only after score-lock	—	—	TEXT																				
	applied_at	Application timestamp	Tracks application date	—	—	TIMESTAMPTZ																				
	updated_at	Last status/update time	Lifecycle tracking	—	—	TIMESTAMPTZ																				
																										
9	job_opening_stages						"PRE_SCREEN
ASSESSMENT
PROBLEM_SOLVING
LIVE_INTERVIEW
HR_INTERVIEW"																			
	Column	Definition	Purpose	PK	FK	Data Type	"stage_type = LIVE_INTERVIEW
stage_name = ""Technical Interview - Round 1"""																			
	stage_id	Unique stage identifier	Identifies a stage within an opening	PK	—	UUID																				
	job_opening_id	Job opening containing the stage	Associates stage with an opening	—	FK → job_openings.job_opening_id	UUID																				
	stage_name	Custom stage name	Allows names such as "Online Assessment", "Live Interview 1", etc.	—	—	VARCHAR(100)																				
	stage_type	Functional type of stage	Determines how the stage behaves	—	—	VARCHAR(30)																				
	stage_order	Position of stage in workflow	Determines candidate progression	—	—	SMALLINT																				
	mode	How the stage is conducted	Online, offline	—	—	VARCHAR(20)																				
	instructions	Candidate instructions/readiness information	Displays preparation/instructions before stage	—	—	TEXT																				
	weight	Stage weightage out of 100 (all stages sum to 100, rank-only)	Stage weightage out of 100 (all stages sum to 100, rank-only)	—	—	SMALLINT																				
	interviewer_access_instructions	Stage-specific, non-sensitive instructions shown to assigned interviewers	—	—	TEXT																					
	max_score	Maximum achievable score	Defines total stage score	—	—	NUMERIC(6,2)																				
	passing_score	Minimum score required	Defines stage qualification threshold	—	—	NUMERIC(6,2)																				
	opening_version_id	Immutable opening version containing this stage	—	FK → opening_versions.opening_version_id	UUID																					
	required_interviewer_count	Required panel size for a Live Interview stage	—	—	SMALLINT																					
	evaluator_aggregation_policy	Approved rule for displaying/combining interviewer evaluations	—	—	VARCHAR(30)																					
	created_at	Stage creation timestamp	Lifecycle tracking	—	—	TIMESTAMPTZ																				
	updated_at	Last stage modification timestamp	Lifecycle tracking	—	—	TIMESTAMPTZ																				
																										
10	stage_results						"application_id = APP001
stage_id       = STAGE001
score          = 82
result         = QUALIFIED
comment        = ""Candidate meets the required experience."""																			
	Column	Purpose	PK	FK	Data Type																					
	stage_result_id	Stage-execution outcome identifier	PK	—	UUID																					
	application_id	Candidate application	—	FK → job_applications.application_id	UUID																					
	stage_id	Completed stage	—	FK → job_opening_stages.stage_id	UUID																					
	attempt_id	Relevant attempt, when applicable	—	FK → stage_attempts.attempt_id	UUID																					
	score	Execution/evaluation score where applicable	—	—	NUMERIC(6,2)																					
	result	COMPLETED, INCOMPLETE, CANCELLED, or EVALUATION_READY	—	—	VARCHAR(30)																					
	result_summary	Non-decision system/evaluation summary	—	—	TEXT																					
	completed_at	Completion time	—	—	TIMESTAMPTZ																					
	created_at	Record creation time	—	—	TIMESTAMPTZ																					
																										
11	stage_sections						"Online Assessment
   │
   ├── Reading Passage
   ├── Question & Answer
   └── Communication"																			
	Column	Purpose	PK	FK	Data Type																					
	section_id	Identifies an assessment section	PK	—	UUID																					
	stage_id	Associates section with stage	—	FK → job_opening_stages.stage_id	UUID																					
	section_name	Identifies section	—	—	VARCHAR(100)																					
	section_type	Defines section behavior	—	—	VARCHAR(30)																					
	section_order	Controls presentation order	—	—	SMALLINT																					
	instructions	Guides applicant	—	—	TEXT																					
	max_score	Defines section scoring	—	—	NUMERIC(6,2)																					
	weight	Section contribution to the stage score	—	—	SMALLINT																					
	preparation_minutes	Candidate preparation time	—	—	INTEGER																					
	duration_minutes	Section time limit	—	—	INTEGER																					
	recording_enabled	Whether the section records audio/video where applicable	—	—	BOOLEAN																					
	re_record_enabled	Whether re-recording is permitted	—	—	BOOLEAN																					
	max_re_record_count	Maximum permitted re-recordings	—	—	SMALLINT																					
																										
12	stage_questions																									
	Column	Purpose	PK	FK	Data Type																					
	question_id	Identifies question	PK	—	UUID																					
	section_id	Associates question with section	—	FK → stage_sections.section_id	UUID																					
	question_text	Displays question	—	—	TEXT																					
	question_type	MCQ, text, audio	—	—	VARCHAR(30)																					
	question_order	Controls display order	—	—	SMALLINT																					
	max_score	Defines question weighting	—	—	NUMERIC(6,2)																					
	help_text	Candidate guidance for the question	—	—	TEXT																					
	is_required	Whether an answer is mandatory	—	—	BOOLEAN																					
	validation_rule	Answer format/range/validation rule	—	—	TEXT																					
																										
13	stage_attempts																									
	Column	Purpose	PK	FK	Data Type																					
	attempt_id	Identifies one candidate attempt	PK	—	UUID																					
	application_id	Identifies candidate	—	FK → job_applications.application_id	UUID																					
	stage_id	Identifies stage	—	FK → job_opening_stages.stage_id	UUID																					
	attempt_number	Supports approved retests	—	—	SMALLINT																					
	opening_version_id	Configuration version used for this attempt	—	FK → opening_versions.opening_version_id	UUID																					
	scheduled_start_at	Scheduled start time where applicable	—	—	TIMESTAMPTZ																					
	deadline_at	Attempt deadline	—	—	TIMESTAMPTZ																					
	permitted_duration_minutes	Time allowed for the attempt	—	—	INTEGER																					
	configuration_version	Opening-version number at attempt start	—	—	INTEGER																					
	configuration_locked_at	When the applicant’s assessment configuration was frozen for this attempt	—	—	TIMESTAMPTZ																					
	readiness_status	NOT_REQUIRED, PENDING, CONFIRMED, or FAILED	—	—	VARCHAR(20)																					
	readiness_confirmed_at	Applicant readiness/consent confirmation timestamp	—	—	TIMESTAMPTZ																					
	interrupted_at	Interruption/inactivity timestamp	—	—	TIMESTAMPTZ																					
	interruption_resolution_due_at	One-minute deadline to submit the packet or grievance	—	—	TIMESTAMPTZ																					
	locked_at	Final packet/attempt lock timestamp	—	—	TIMESTAMPTZ																					
	terminal_outcome	COMPLETED_SUCCESSFUL, COMPLETED_UNSUCCESSFUL, TEST_STARTED_NOT_COMPLETED, VALID_WITHDRAWAL_OR_INCOMPLETE, or CANCELLED	—	—	VARCHAR(50)																					
	submitted_at	Records completion	—	—	TIMESTAMPTZ																					
																										
14	candidate_responses																									
	Column	Purpose	PK	FK	Data Type																					
	response_id	Candidate response identifier	PK	—	UUID																					
	attempt_id	Parent stage attempt	—	FK → stage_attempts.attempt_id	UUID																					
	question_id	Answered question	—	FK → stage_questions.question_id	UUID																					
	response_text	Written answer or active-recording transcript	—	—	TEXT																					
	selected_option_id	Selected MCQ option, where applicable	—	FK → stage_question_options.option_id	UUID																					
	response_status	DRAFT, SUBMITTED, or AUTO_SUBMITTED	—	—	VARCHAR(30)																					
	active_media_recording_id	Recording version selected for analysis	—	FK → candidate_response_media_versions.media_recording_id	UUID																					
	locked_at	Final-answer lock time	—	—	TIMESTAMPTZ																					
	responded_at	Response time	—	—	TIMESTAMPTZ																					
																										
15	ai_stage_evaluations																									
	Column	Purpose	PK	FK	Data Type																					
	evaluation_id	Identifies one AI evaluation run	PK	—	UUID																					
	attempt_id	Candidate stage attempt being evaluated, including pre-screen attempt	—	FK → stage_attempts.attempt_id	UUID																					
	evaluation_type	PRE_SCREEN, ASSESSMENT, VOICE_ANALYSIS	—	—	VARCHAR(30)																					
	evaluation_config_version	Version of the approved criteria/prompt configuration used	—	—	INTEGER																					
	processing_status	PENDING, PROCESSING, COMPLETED, FAILED	—	—	VARCHAR(20)																					
	processing_error	Failure detail; populated only when processing fails	—	—	TEXT																					
	input_document_id	Résumé/document analysed for pre-screening; otherwise NULL	—	FK → application_documents.application_document_id	UUID																					
	retry_of_evaluation_id	Earlier failed evaluation retried by this row	—	FK → ai_stage_evaluations.evaluation_id	UUID																					
	score	Optional AI numerical score	—	—	NUMERIC(6,2)																					
	evaluation	Immutable strengths, weaknesses, evidence, discrepancies, and overall analysis	—	—	TEXT																					
	created_at	Time the evaluation job was created	—	—	TIMESTAMPTZ																					
	completed_at	Time AI processing completed successfully or failed	—	—	TIMESTAMPTZ																					
																										
16	hr_decisions																									
	Column	Purpose	PK	FK	Data Type																					
	decision_id	Identifies HR action	PK	—	UUID																					
	application_id	Identifies candidate	—	FK → job_applications.application_id	UUID																					
	stage_id	Identifies stage	—	FK → job_opening_stages.stage_id	UUID																					
	decided_by_membership_id	Identifies reviewer	—	FK → organization_memberships.membership_id	UUID																					
	ai_evaluation_id	Pins the exact immutable AI result considered by HR	—	FK → ai_stage_evaluations.evaluation_id	UUID																					
	decision	QUALIFIED_FOR_ASSESSMENT, PRE_SCREEN_PENDING, NOT_CURRENTLY_QUALIFIED, LIVE_INTERVIEW_RECOMMENDED, LIVE_INTERVIEW_HOLD, ADVANCED, HOLD, REJECTED, SELECTED, or NOT_SELECTED	—	—	VARCHAR(40)																					
	created_at	Audit/history	—	—	TIMESTAMPTZ																					
																										
17	assessment_grievances																									
	Column	Purpose	PK	FK	Data Type																					
	grievance_id	Identifies grievance	PK	—	UUID																					
	application_id	Identifies candidate/application	—	FK → job_applications.application_id	UUID																					
	stage_id	Identifies affected stage	—	FK → job_opening_stages.stage_id	UUID																					
	attempt_id	Specific assessment/interview attempt affected by the grievance	—	FK → stage_attempts.attempt_id	UUID																					
	decision_due_at	24-hour grievance-window deadline	—	—	TIMESTAMPTZ																					
	confidential_hr_note	HR-only decision rationale	—	—	TEXT																					
	comment	Records explanation	—	—	TEXT																					
	status	PENDING, APPROVED_FOR_REATTEMPT, REJECTED, EXPIRED, or CANCELLED	—	—	VARCHAR(30)																					
	decided_by_membership_id	Identifies HR decision maker	—	FK → organization_memberships.membership_id	UUID																					
	created_at	Audit tracking	—	—	TIMESTAMPTZ																					
	decided_at	Audit tracking	—	—	TIMESTAMPTZ																					
																										
18	job_requisitions																									
	Column	Purpose	PK	FK	Data Type																					
	requisition_id	Identifies each hiring request	PK	—	UUID																					
	organization_id	Enforces tenant isolation	—	FK → organizations.organization_id	UUID																					
	created_by_membership_id	Identifies the requisition owner	—	FK → organization_memberships.membership_id	UUID																					
	title	Identifies the hiring need	—	—	VARCHAR(150)																					
	description	Describes the requested position	—	—	TEXT																					
	position_count	Defines approved headcount	—	—	INTEGER																					
	business_justification	Supports HR approval decision	—	—	TEXT																					
	salary_min	Defines approved salary range	—	—	NUMERIC(12,2)																					
	salary_max	Defines approved salary range	—	—	NUMERIC(12,2)																					
	experience_required	Defines candidate requirement	—	—	TEXT																					
	skills	Defines candidate requirement	—	—	TEXT																					
	job_requirements	Defines candidate requirement	—	—	TEXT																					
	eligibility_criteria	Defines pre-screen eligibility	—	—	TEXT																					
	employment_type	Full-time, contract, internship, etc.	—	—	VARCHAR(30)																					
	location_work_mode	Remote, hybrid, or onsite	—	—	VARCHAR(50)																					
	status	DRAFT, PENDING_APPROVAL, RETURNED, APPROVED, or WITHDRAWN	—	—	VARCHAR(30)																					
	department	Requesting department	—	—	VARCHAR(100)																					
	target_location	Requested work location, separate from work mode	—	—	VARCHAR(150)																					
	target_start_date	Requested joining/start date	—	—	DATE																					
	approved_requisition_version_id	Immutable approved version that may create an opening	—	FK → requisition_versions.requisition_version_id	UUID																					
	submitted_at	Tracks first submission	—	—	TIMESTAMPTZ																					
	last_submitted_at	Supports resubmission ordering	—	—	TIMESTAMPTZ																					
	created_at	Lifecycle tracking	—	—	TIMESTAMPTZ																					
	updated_at	Lifecycle tracking	—	—	TIMESTAMPTZ																					
																										
19	requisition_reviews																									
	Column	Purpose	PK	FK	Data Type																					
	review_id	Identifies each HR review	PK	—	UUID																					
	requisition_id	Links review to requisition	—	FK → job_requisitions.requisition_id	UUID																					
	reviewed_by_membership_id	Identifies the reviewer	—	FK → organization_memberships.membership_id	UUID																					
	review_status	APPROVED, REJECTED, or RETURNED	—	—	VARCHAR(30)																					
	comment	Required for rejection or return	—	—	TEXT																					
	requisition_version_id	Exact requisition version reviewed by HR	—	FK → requisition_versions.requisition_version_id	UUID																					
	last_submitted_at	—	—	—	TIMESTAMPTZ																					
	created_at	—	—	—	TIMESTAMPTZ																					
	updated_at	—	—	—	TIMESTAMPTZ																					
																										
20	position_fills																									
	Column	Purpose	PK	FK	Data Type																					
	position_fill_id	Identifies each filled position	PK	—	UUID																					
	job_opening_id	Links fill to opening	—	FK → job_openings.job_opening_id	UUID																					
	application_id	Links the fill to a candidate when applicable	—	FK → job_applications.application_id	UUID																					
	filled_by_membership_id	Identifies the authorized actor	—	FK → organization_memberships.membership_id	UUID																					
	filled_at	Tracks headcount fulfillment	—	—	TIMESTAMPTZ																					
	reason	Supports verification and audit	—	—	TEXT																					
	selection_id	HR-authorised selection that produced this fill	—	FK → application_selections.selection_id	UUID																					
	created_at	Record creation timestamp	—	—	TIMESTAMPTZ																					
																										
21	ai_evaluation_areas																									
	Column	Purpose	PK	FK	Data Type																					
	area_evaluation_id	Identifies each per-area score	PK	—	UUID																					
	evaluation_id	Parent AI evaluation	—	FK → ai_stage_evaluations.evaluation_id	UUID																					
	area_name	Area label (Task, Thinking, Follow-through, Learning, Judgment, People, Integrity)	—	—	VARCHAR(40)																					
	area_score	1–4 score for the area	—	—	SMALLINT																					
	reasoning	AI reasoning for the area score	—	—	TEXT																					
																										
22	voice_integrity_checks																									
	Column	Purpose	PK	FK	Data Type																					
	integrity_check_id	Identifies each voice integrity check	PK	—	UUID																					
	attempt_id	Stage attempt the check applies to	—	FK → stage_attempts.attempt_id	UUID																					
	comparison	What was compared (answer-vs-reading, answer-vs-intro)	—	—	VARCHAR(30)																					
	flag	NONE, REVIEW, or NOT_APPLICABLE for an approved typed accommodation	—	—	VARCHAR(20)																					
	disposition	Resolution of the flag (dismissed, confirmed)	—	—	VARCHAR(30)																					
	resolved_by_membership_id	Who resolved the flag (nullable)	—	FK → organization_memberships.membership_id	UUID																					
	created_at	Check timestamp	—	—	TIMESTAMPTZ																					
	resolved_at	Resolution timestamp	—	—	TIMESTAMPTZ																					
																										
23	interview_criteria																									
	Column	Purpose	PK	FK	Data Type																					
	criteria_id	Identifies each scoring criterion	PK	—	UUID																					
	stage_id	Stage the criterion belongs to	—	FK → job_opening_stages.stage_id	UUID																					
	criteria_name	Criterion label (Technical, Behavioral, Communication, etc.)	—	—	VARCHAR(100)																					
	weight	Criterion weightage out of 100 (criteria in a stage sum to 100)	—	—	SMALLINT																					
	max_score	Maximum score for the criterion	—	—	NUMERIC(6,2)																					
	guidance	Role-specific guidance/evidence expected for this criterion	—	—	TEXT																					
	is_required	Whether the interviewer must score this criterion before submission	—	—	BOOLEAN																					
	rating_scale	Approved labels/range for the criterion, where different from max_score	—	—	JSONB																					
	criteria_order	Display order of the criterion	—	—	SMALLINT																					
	created_at	Creation timestamp	—	—	TIMESTAMPTZ																					
	updated_at	Last modification timestamp	—	—	TIMESTAMPTZ																					
																										
24	interview_results																									
	Column	Purpose	PK	FK	Data Type																					
	interview_result_id	Identifies each per-criterion score	PK	—	UUID																					
	criteria_id	Criterion being scored	—	FK → interview_criteria.criteria_id	UUID																					
	stage_interviewer_id	Assigned interviewer and interview-attempt link	—	FK → stage_interviewers.stage_interviewer_id	UUID																					
	score	Score awarded for the criterion (out of max_score)	—	—	NUMERIC(6,2)																					
	comment	Interviewer’s criterion-specific evidence/comment	—	—	TEXT																					
	result_status	DRAFT or SUBMITTED; submitted rows are immutable	—	—	VARCHAR(20)																					
	saved_at	Last draft-save timestamp	—	—	TIMESTAMPTZ																					
	updated_at	Last update; blocked after submission	—	—	TIMESTAMPTZ																					
																										
25	stage_interviewers																									
	Column	Purpose	PK	FK	Data Type																					
	stage_interviewer_id	Identifies each interviewer assignment to an applicant's stage evaluation	PK	—	UUID																					
	application_id	Identifies the candidate/application being interviewed	—	FK → job_applications.application_id	UUID																					
	stage_id	Identifies the stage (e.g. Live Interview) the interviewer is assigned to	—	FK → job_opening_stages.stage_id	UUID																					
	interviewer_membership_id	Identifies the assigned interviewer (organization member)	—	FK → organization_memberships.membership_id	UUID																					
	attempt_id	Exact scheduled/retake interview attempt	—	FK → stage_attempts.attempt_id	UUID																					
	replaced_by_stage_interviewer_id	Replacement interviewer assignment, if HR authorises a retake	—	FK → stage_interviewers.stage_interviewer_id	UUID																					
	evaluation_started_at	When the assigned interviewer starts the evaluation	—	—	TIMESTAMPTZ																					
	submission_locked_at	When submitted evaluation became immutable	—	—	TIMESTAMPTZ																					
	access_revoked_at	When future evaluation access was revoked	—	—	TIMESTAMPTZ																					
	access_revocation_reason	Reason for revocation/replacement	—	—	TEXT																					
	status	PENDING, IN_PROGRESS, COMPLETED, REPLACED, REVOKED, or CANCELLED	—	—	VARCHAR(30)																					
	overall_comment	Interviewer's free-text comment on the interview	—	—	TEXT																					
	interview_attempt_number	Supports retry/replacement panels	—	—	SMALLINT																					
	assigned_at	Interviewer-assignment timestamp	—	—	TIMESTAMPTZ																					
	submitted_at	When the interviewer submitted the evaluation; NULL until submitted, immutable after	—	—	TIMESTAMPTZ																					
																										
26	applicant_profiles																									
	Column	Purpose	PK	FK	Data Type																					
	applicant_profile_id	Identifies the applicant profile	PK	—	UUID																					
	canonical_person_id	Person who owns this profile	—	→ canonical_people.canonical_person_id	UUID																					
	phone	Applicant contact number	—	—	VARCHAR(20)																					
	college_name	College or institution	—	—	VARCHAR(200)																					
	degree_name	Degree/course name	—	—	VARCHAR(150)																					
	specialization	Subject/branch/specialisation	—	—	VARCHAR(150)																					
	cgpa	CGPA or equivalent score	—	—	NUMERIC(4,2)																					
	graduation_year	Graduation year	—	—	SMALLINT																					
	city	Applicant city, if collected	—	—	VARCHAR(100)																					
	profile_status	ACTIVE, DEACTIVATED	—	—	VARCHAR(20)																					
	created_at	Profile creation time	—	—	TIMESTAMPTZ																					
	updated_at	Last profile update time	—	—	TIMESTAMPTZ																					
																										
27	stage_question_options																									
	Column	Purpose	PK	FK	Data Type																					
	option_id	Identifies one answer option	PK	—	UUID																					
	question_id	Parent MCQ question	—	→ stage_questions.question_id	UUID																					
	option_text	Option shown to the applicant	—	—	TEXT																					
	option_order	Display order	—	—	SMALLINT																					
	is_correct	Correct option for auto-scored assessment questions	—	—	BOOLEAN																					
	score_value	Marks awarded for this option	—	—	NUMERIC(6,2)																					
	created_at	Creation timestamp	—	—	TIMESTAMPTZ																					
																										
28	requisition_versions																									
	Column	Purpose	PK	FK	Data Type																					
	requisition_version_id	Immutable requisition snapshot	PK	—	UUID																					
	requisition_id	Parent requisition	—	FK → job_requisitions.requisition_id	UUID																					
	version_number	Sequential version within requisition	—	—	INTEGER																					
	title	Snapshotted job title	—	—	VARCHAR(150)																					
	department	Snapshotted requesting department	—	—	VARCHAR(100)																					
	description	Snapshotted role description	—	—	TEXT																					
	target_location	Snapshotted requested location	—	—	VARCHAR(150)																					
	target_start_date	Snapshotted requested joining date	—	—	DATE																					
	position_count	Snapshotted headcount	—	—	INTEGER																					
	business_justification	Snapshotted business case	—	—	TEXT																					
	salary_min	Snapshotted salary lower bound	—	—	NUMERIC(12,2)																					
	salary_max	Snapshotted salary upper bound	—	—	NUMERIC(12,2)																					
	experience_required	Snapshotted experience requirement	—	—	TEXT																					
	skills	Snapshotted required skills	—	—	TEXT																					
	job_requirements	Snapshotted job requirements	—	—	TEXT																					
	eligibility_criteria	Snapshotted pre-screen criteria	—	—	TEXT																					
	employment_type	Snapshotted employment type	—	—	VARCHAR(30)																					
	location_work_mode	Snapshotted work mode	—	—	VARCHAR(50)																					
	submitted_by_membership_id	Hiring Manager who submitted this version	—	FK → organization_memberships.membership_id	UUID																					
	submitted_at	Version-lock timestamp	—	—	TIMESTAMPTZ																					
	created_at	Snapshot creation timestamp	—	—	TIMESTAMPTZ																					
																										
29	requisition_version_stages																									
	Column	Purpose	PK	FK	Data Type																					
	requisition_version_stage_id	Configured requisition-stage identifier	PK	—	UUID																					
	requisition_version_id	Immutable requisition version	—	FK → requisition_versions.requisition_version_id	UUID																					
	stage_name	Applicant-facing stage name	—	—	VARCHAR(100)																					
	stage_type	PRE_SCREEN, ASSESSMENT, LIVE_INTERVIEW, HR_INTERVIEW, or CUSTOM	—	—	VARCHAR(30)																					
	stage_order	Stage sequence	—	—	SMALLINT																					
	mode	ONLINE, IN_PERSON, REMOTE_MEETING, or configured mode	—	—	VARCHAR(30)																					
	instructions	Applicant-facing instructions	—	—	TEXT																					
	weight	Stage weight; all stages total 100	—	—	SMALLINT																					
	max_score	Configured total marks	—	—	NUMERIC(6,2)																					
	passing_score	Configured benchmark; never an automatic decision	—	—	NUMERIC(6,2)																					
	conductor_rule	Role or participant rule for conducting the stage	—	—	TEXT																					
	required_interviewer_count	Required evaluators/panel size	—	—	SMALLINT																					
	requires_scheduling	Whether the stage requires scheduling	—	—	BOOLEAN																					
	requires_applicant_submission	Whether applicant submission is required	—	—	BOOLEAN																					
	requires_evaluator_submission	Whether evaluator submission is required	—	—	BOOLEAN																					
	completion_condition	Rule used to mark stage completion	—	—	TEXT																					
	created_at	Configuration creation timestamp	—	—	TIMESTAMPTZ																					
																										
30	requisition_version_stage_sections																									
	Column	Purpose	PK	FK	Data Type																					
	requisition_version_section_id	Configured section identifier	PK	—	UUID																					
	requisition_version_stage_id	Parent requisition stage	—	FK → requisition_version_stages.requisition_version_stage_id	UUID																					
	section_name	Section name	—	—	VARCHAR(100)																					
	section_type	Section behaviour/type	—	—	VARCHAR(30)																					
	section_order	Presentation order	—	—	SMALLINT																					
	instructions	Candidate instructions	—	—	TEXT																					
	weight	Section contribution to stage score	—	—	SMALLINT																					
	max_score	Section maximum score	—	—	NUMERIC(6,2)																					
	preparation_minutes	Candidate preparation time	—	—	INTEGER																					
	duration_minutes	Section time limit	—	—	INTEGER																					
	recording_enabled	Whether recording is enabled	—	—	BOOLEAN																					
	re_record_enabled	Whether re-recording is enabled	—	—	BOOLEAN																					
	max_re_record_count	Maximum re-recordings	—	—	SMALLINT																					
																										
31	requisition_version_stage_questions																									
	Column	Purpose	PK	FK	Data Type																					
	requisition_version_question_id	Configured question identifier	PK	—	UUID																					
	requisition_version_section_id	Parent requisition section	—	FK → requisition_version_stage_sections.requisition_version_section_id	UUID																					
	question_text	Question shown to the applicant	—	—	TEXT																					
	question_type	MCQ, text, audio, video, or configured type	—	—	VARCHAR(30)																					
	question_order	Display order	—	—	SMALLINT																					
	help_text	Candidate guidance	—	—	TEXT																					
	is_required	Whether the answer is mandatory	—	—	BOOLEAN																					
	validation_rule	Answer validation rule	—	—	TEXT																					
	max_score	Question maximum score	—	—	NUMERIC(6,2)																					
																										
																										
32	opening_versions																									
	Column	Purpose	PK	FK	Data Type																					
	opening_version_id	Immutable opening-package identifier	PK	—	UUID																					
	job_opening_id	Parent opening	—	FK → job_openings.job_opening_id	UUID																					
	requisition_version_id	Approved source requisition version	—	FK → requisition_versions.requisition_version_id	UUID																					
	version_number	Sequential opening version	—	—	INTEGER																					
	based_on_opening_version_id	Source version for an approved revision	—	FK → opening_versions.opening_version_id	UUID																					
	title	Snapshotted title	—	—	VARCHAR(150)																					
	description	Snapshotted description	—	—	TEXT																					
	target_headcount	Snapshotted target headcount	—	—	INTEGER																					
	work_location	Snapshotted location	—	—	VARCHAR(150)																					
	target_start_date	Snapshotted joining date	—	—	DATE																					
	version_status	DRAFT, PENDING_OPENING_APPROVAL, RETURNED, APPROVED_TO_PUBLISH, PUBLISHED, or REJECTED	—	—	VARCHAR(30)																					
	submitted_by_membership_id	Hiring Manager who submitted it	—	FK → organization_memberships.membership_id	UUID																					
	submitted_at	Submission time	—	—	TIMESTAMPTZ																					
	published_at	Activation time	—	—	TIMESTAMPTZ																					
	created_at	Creation time	—	—	TIMESTAMPTZ																					
	updated_at	Last draft-update time	—	—	TIMESTAMPTZ																					
																										
33	opening_stage_interviewer_nominations																									
	Column	Purpose	PK	FK	Data Type																					
	nomination_id	Proposed interviewer identifier	PK	—	UUID																					
	opening_version_id	Opening version containing the nomination	—	FK → opening_versions.opening_version_id	UUID																					
	stage_id	Configured Live Interview stage	—	FK → job_opening_stages.stage_id	UUID																					
	interviewer_membership_id	Existing member, when matched	—	FK → organization_memberships.membership_id	UUID																					
	invitee_name	Nominated interviewer's name	—	—	VARCHAR(200)																					
	invitee_email	Nominated interviewer's email	—	—	VARCHAR(255)																					
	nomination_status	NOMINATED, HR_APPROVED, INVITED, ACTIVATED, EXPIRED, REPLACED, or REVOKED	—	—	VARCHAR(30)																					
	hr_reviewed_by_membership_id	HR reviewer who approved/returned the nomination	—	FK → organization_memberships.membership_id	UUID																					
	hr_reviewed_at	HR decision timestamp	—	—	TIMESTAMPTZ																					
	invitation_token_reference	Opaque one-time invitation-token reference	—	—	VARCHAR(255)																					
	normalized_invitee_email	Normalized email used to prevent duplicate nominations per opening/stage	—	—	VARCHAR(255)																					
	invitation_sent_at	Invitation send timestamp	—	—	TIMESTAMPTZ																					
	invitation_expires_at	24-hour invitation expiry	—	—	TIMESTAMPTZ																					
	activated_at	Successful registration/role-activation timestamp	—	—	TIMESTAMPTZ																					
	replaced_by_nomination_id	Replacement nomination where applicable	—	FK → opening_stage_interviewer_nominations.nomination_id	UUID																					
	created_by_membership_id	Nominating Hiring Manager	—	FK → organization_memberships.membership_id	UUID																					
	created_at	Nomination timestamp	—	—	TIMESTAMPTZ																					
																										
34	hiring_manager_comments																									
	Column	Purpose	PK	FK	Data Type																					
	hiring_manager_comment_id	Hiring Manager observation identifier	PK	—	UUID																					
	application_id	Applicant being commented on	—	FK → job_applications.application_id	UUID																					
	stage_id	Relevant stage; nullable for opening-level applicant comment	—	FK → job_opening_stages.stage_id	UUID																					
	author_membership_id	Assigned Hiring Manager who wrote the comment	—	FK → organization_memberships.membership_id	UUID																					
	comment	Hiring Manager observation; never an HR decision	—	—	TEXT																					
	created_at	Comment timestamp	—	—	TIMESTAMPTZ																					
																										
35	workflow_action_requests																									
	Column	Purpose	PK	FK	Data Type																					
	action_request_id	Requisition withdrawal-request identifier	PK	—	UUID																					
	requisition_id	Nullable; target requisition when action is WITHDRAW	—	FK → job_requisitions.requisition_id	UUID																					
	requested_by_membership_id	HR requester	—	FK → organization_memberships.membership_id	UUID																					
	target_membership_id	Creating Hiring Manager who may execute/reject the request	—	FK → organization_memberships.membership_id	UUID																					
	action_type	WITHDRAW, SUSPEND, REACTIVATE, CLOSE, or APPLICATION_REACTIVATE	—	—	VARCHAR(40)																					
	reason	Reason for the request	—	—	TEXT																					
	status	PENDING, ACCEPTED, REJECTED, CANCELLED, or EXPIRED	—	—	VARCHAR(30)																					
	job_opening_id	Nullable; target opening for SUSPEND, REACTIVATE, or CLOSE	—	FK → job_openings.job_opening_id	UUID																					
	application_id	Nullable; target application for a requested reactivation	—	FK → job_applications.application_id	UUID																					
	responded_by_membership_id	Hiring Manager who accepted/rejected	—	FK → organization_memberships.membership_id	UUID																					
	responded_at	Response timestamp	—	—	TIMESTAMPTZ																					
	created_at	Request timestamp	—	—	TIMESTAMPTZ																					
	updated_at	Last status-update timestamp	—	—	TIMESTAMPTZ																					
																										
36	canonical_people																									
	Column	Purpose	PK	FK	Data Type																					
	canonical_person_id	Verified person identifier shared across organisation realms	PK	—	UUID																					
	normalized_email	Canonical verified email address	—	—	VARCHAR(255)																					
	email_verified_at	Email-verification timestamp	—	—	TIMESTAMPTZ																					
	created_at	Canonical-person creation timestamp	—	—	TIMESTAMPTZ																					
	updated_at	Last identity update timestamp	—	—	TIMESTAMPTZ																					
																										
37	application_documents																									
	Column	Purpose	PK	FK	Data Type																					
	application_document_id	Application document identifier	PK	—	UUID																					
	application_id	Applicant/application that owns the document	—	FK → job_applications.application_id	UUID																					
	document_type	RESUME, COVER_LETTER, PORTFOLIO, CERTIFICATE, or configured document type	—	—	VARCHAR(40)																					
	storage_object_key	Private object-storage reference; never a public URL	—	—	TEXT																					
	file_name	Original upload filename	—	—	VARCHAR(255)																					
	mime_type	Uploaded file content type	—	—	VARCHAR(100)																					
	file_size_bytes	Uploaded file size	—	—	BIGINT																					
	content_hash	File integrity hash	—	—	VARCHAR(128)																					
	uploaded_at	Upload timestamp	—	—	TIMESTAMPTZ																					
	deleted_at	Deletion timestamp after retention expiry	—	—	TIMESTAMPTZ																					
																										
38	application_stage_transitions																									
	Column	Purpose	PK	FK	Data Type																					
	transition_id	Application stage-history identifier	PK	—	UUID																					
	application_id	Application whose lifecycle changed	—	FK → job_applications.application_id	UUID																					
	from_stage_id	Previous stage; NULL for initial submission	—	FK → job_opening_stages.stage_id	UUID																					
	to_stage_id	New/current stage; NULL for terminal closure	—	FK → job_opening_stages.stage_id	UUID																					
	from_status	Previous application status	—	—	VARCHAR(40)																					
	to_status	New application status	—	—	VARCHAR(40)																					
	transition_type	SUBMITTED, HR_DECISION, INVITED, STAGE_STARTED, STAGE_COMPLETED, SUSPENDED, SELECTED, ONBOARDING, or CLOSED	—	—	VARCHAR(40)																					
	hr_decision_id	Decision causing the transition where applicable	—	FK → hr_decisions.decision_id	UUID																					
	performed_by_membership_id	HR actor; NULL for trusted system action	—	FK → organization_memberships.membership_id	UUID																					
	occurred_at	Transition timestamp	—	—	TIMESTAMPTZ																					
																										
39	assessment_invitations																									
	Column	Purpose	PK	FK	Data Type																					
	invitation_id	Secure assessment-invitation identifier	PK	—	UUID																					
	application_id	Applicant receiving the invitation	—	FK → job_applications.application_id	UUID																					
	attempt_id	Scheduled attempt enabled by the invitation	—	FK → stage_attempts.attempt_id	UUID																					
	recipient_email	Verified delivery address	—	—	VARCHAR(255)																					
	token_reference	Opaque, single-use access-token reference	—	—	VARCHAR(255)																					
	status	PENDING, SENT, OPENED, REDEEMED, EXPIRED, CANCELLED, or FAILED	—	—	VARCHAR(30)																					
	sent_at	Invitation send timestamp	—	—	TIMESTAMPTZ																					
	expires_at	Invitation expiry timestamp	—	—	TIMESTAMPTZ																					
	redeemed_at	Successful token-redemption timestamp	—	—	TIMESTAMPTZ																					
																										
40	pre_screen_decision_batches																									
	Column	Purpose	PK	FK	Data Type																					
	decision_batch_id	Controlled HR bulk pre-screen action identifier	PK	—	UUID																					
	organization_id	Tenant boundary for the batch	—	FK → organizations.organization_id	UUID																					
	job_opening_id	One opening permitted per batch	—	FK → job_openings.job_opening_id	UUID																					
	opening_version_id	One configuration version permitted per batch	—	FK → opening_versions.opening_version_id	UUID																					
	stage_id	Pre-screen stage covered by the batch	—	FK → job_opening_stages.stage_id	UUID																					
	decision	Requested individual decision for each selected application	—	—	VARCHAR(40)																					
	initiated_by_membership_id	HR Reviewer initiating the operation	—	FK → organization_memberships.membership_id	UUID																					
	idempotency_key	Prevents duplicate batch processing	—	—	VARCHAR(255)																					
	status	DRAFT, VALIDATING, COMPLETED, REJECTED, or ROLLED_BACK	—	—	VARCHAR(30)																					
	confirmed_at	HR attestation/confirmation time	—	—	TIMESTAMPTZ																					
	completed_at	Atomic batch-completion time	—	—	TIMESTAMPTZ																					
	created_at	Batch creation timestamp	—	—	TIMESTAMPTZ																					
																										
41	hr_decision_internal_notes																									
	Column	Purpose	PK	FK	Data Type																					
	internal_note_id	Confidential HR note identifier	PK	—	UUID																					
	decision_id	Exactly one internal note for the HR decision	—	FK → hr_decisions.decision_id	UUID																					
	author_membership_id	HR Reviewer who wrote the note	—	FK → organization_memberships.membership_id	UUID																					
	note	Internal rationale; never returned to non-HR APIs	—	—	TEXT																					
	created_at	Note timestamp	—	—	TIMESTAMPTZ																					
																										
42	hr_decision_status_messages																									
	Column	Purpose	PK	FK	Data Type																					
	status_message_id	Applicant-visible decision message identifier	PK	—	UUID																					
	decision_id	Exactly one status message for the HR decision	—	FK → hr_decisions.decision_id	UUID																					
	message	Status-only applicant communication	—	—	TEXT																					
	created_at	Message timestamp	—	—	TIMESTAMPTZ																					
																										
43	application_accommodations																									
	Column	Purpose	PK	FK	Data Type																					
	accommodation_id	HR-restricted accommodation record identifier	PK	—	UUID																					
	application_id	Applicant requesting accommodation	—	FK → job_applications.application_id	UUID																					
	stage_id	Affected stage	—	FK → job_opening_stages.stage_id	UUID																					
	request_detail	Restricted accommodation request	—	—	TEXT																					
	operational_instruction	Minimum non-sensitive instruction supplied to an evaluator	—	—	TEXT																					
	status	PENDING, APPROVED, REJECTED, CANCELLED, or COMPLETED	—	—	VARCHAR(30)																					
	decided_by_membership_id	HR decision-maker	—	FK → organization_memberships.membership_id	UUID																					
	decided_at	Decision timestamp	—	—	TIMESTAMPTZ																					
	created_at	Request timestamp	—	—	TIMESTAMPTZ																					
																										
																										
44	application_selections																									
	Column	Purpose	PK	FK	Data Type																					
	selection_id	Final HR-authorised candidate selection	PK	—	UUID																					
	application_id	Selected application	—	FK → job_applications.application_id	UUID																					
	selected_by_membership_id	Authorised HR selector	—	FK → organization_memberships.membership_id	UUID																					
	selected_at	Selection/reservation timestamp	—	—	TIMESTAMPTZ																					
	offer_status	NOT_RECORDED, OFFERED, ACCEPTED, DECLINED, or WITHDRAWN	—	—	VARCHAR(30)																					
	offer_recorded_by_membership_id	Actor recording offer outcome	—	FK → organization_memberships.membership_id	UUID																					
	offer_recorded_at	Offer outcome timestamp	—	—	TIMESTAMPTZ																					
	reservation_status	RESERVED, RELEASED, or CONSUMED	—	—	VARCHAR(30)																					
	reserved_at	Vacancy-reservation timestamp	—	—	TIMESTAMPTZ																					
	released_at	Reservation-release timestamp	—	—	TIMESTAMPTZ																					
	release_reason	Reason capacity was released	—	—	TEXT																					
																										
45	onboarding_records																									
	Column	Purpose	PK	FK	Data Type																					
	onboarding_id	Recruitment-side onboarding record	PK	—	UUID																					
	selection_id	Final selection being onboarded	—	FK → application_selections.selection_id	UUID																					
	status	ONBOARDING_PENDING, COMPLETED, FAILED, or CANCELLED	—	—	VARCHAR(30)																					
	updated_by_membership_id	HR updater	—	FK → organization_memberships.membership_id	UUID																					
	updated_at	Last onboarding update timestamp	—	—	TIMESTAMPTZ																					
	completion_reference	External onboarding-system reference	—	—	TEXT																					
	failure_reason	Reason for failed/cancelled onboarding	—	—	TEXT																					
																										
46	legal_holds																									
	Column	Purpose	PK	FK	Data Type																					
	legal_hold_id	HR/legal retention-hold identifier	PK	—	UUID																					
	organization_id	Organisation covered by the hold	—	FK → organizations.organization_id	UUID																					
	application_id	Optional application-specific hold	—	FK → job_applications.application_id	UUID																					
	scope	ORGANIZATION or APPLICATION	—	—	VARCHAR(30)																					
	reason	Hold rationale	—	—	TEXT																					
	status	ACTIVE or RELEASED	—	—	VARCHAR(20)																					
	placed_by_membership_id	Authorised HR/legal user who placed the hold	—	FK → organization_memberships.membership_id	UUID																					
	placed_at	Hold timestamp	—	—	TIMESTAMPTZ																					
	released_by_membership_id	Authorised user releasing the hold	—	FK → organization_memberships.membership_id	UUID																					
	released_at	Hold-release timestamp	—	—	TIMESTAMPTZ																					
																										
																										
47	notification_outbox																									
	Column	Purpose	PK	FK	Data Type																					
	notification_id	Reliable notification identifier	PK	—	UUID																					
	organization_id	Tenant boundary	—	FK → organizations.organization_id	UUID																					
	entity_type	REQUISITION, OPENING_VERSION, APPLICATION, GRIEVANCE, INTERVIEW, or ONBOARDING	—	—	VARCHAR(40)																					
	entity_id	Record causing the notification	—	—	UUID																					
	event_type	Supports INTERVIEWER_INVITED, INTERVIEW_SCHEDULED, INTERVIEW_RESCHEDULED, EVALUATION_SUBMITTED, ALL_EVALUATIONS_COMPLETED, INTERVIEWER_REPLACED, and INVITATION_EXPIRED	—	—	VARCHAR(60)																					
	sender_membership_id	Responsible HR actor where applicable	—	FK → organization_memberships.membership_id	UUID																					
	recipient_email	Delivery target	—	—	VARCHAR(255)																					
	correlation_id	Links one notification to a controlled bulk or workflow operation	—	—	UUID																					
	template_data	Non-sensitive template payload	—	—	JSONB																					
	status	PENDING, SENT, FAILED, or RETRYING	—	—	VARCHAR(30)																					
	retry_count	Delivery retry count	—	—	INTEGER																					
	provider_reason	Provider failure detail	—	—	TEXT																					
	created_at	Outbox creation timestamp	—	—	TIMESTAMPTZ																					
	sent_at	Successful delivery timestamp	—	—	TIMESTAMPTZ																					
																										
48	audit_events																									
	Column	Purpose	PK	FK	Data Type																					
	audit_event_id	Append-only workflow audit identifier	PK	—	UUID																					
	organization_id	Tenant-isolated audit history	—	FK → organizations.organization_id	UUID																					
	correlation_id	Links related events from a batch, invitation, retake, or workflow action	—	—	UUID																					
	entity_type	REQUISITION, OPENING, APPLICATION, DECISION, GRIEVANCE, INTERVIEW, or ONBOARDING	—	—	VARCHAR(40)																					
	entity_id	Audited entity	—	—	UUID																					
	actor_membership_id	Responsible actor; NULL for trusted system event	—	FK → organization_memberships.membership_id	UUID																					
	event_type	Supports INTERVIEWER_ASSIGNED, INVITATION_SENT, INTERVIEW_SCHEDULED, EVALUATION_OPENED, EVALUATION_SUBMITTED, INTERVIEW_COMPLETED, INTERVIEWER_REPLACED, RETAKE_SCHEDULED, and ACCESS_REVOKED	—	—	VARCHAR(60)																					
	reason	Return, rejection, exception, or cancellation reason	—	—	TEXT																					
	event_data	Non-sensitive structured metadata	—	—	JSONB																					
	created_at	Append-only event timestamp	—	—	TIMESTAMPTZ																					
																										
																										
																										
49	candidate_response_media_versions																									
	Column	Purpose	PK	FK	Data Type																					
	media_recording_id	Immutable audio-recording version identifier	PK	—	UUID																					
	response_id	Applicant response to which this recording belongs	—	FK → candidate_responses.response_id	UUID																					
	attempt_id	Assessment attempt containing the recording	—	FK → stage_attempts.attempt_id	UUID																					
	recording_version	Monotonically increasing recording/re-record version	—	—	SMALLINT																					
	storage_object_key	Private recording-storage reference; never a public URL	—	—	TEXT																					
	mime_type	Recording media type	—	—	VARCHAR(100)																					
	duration_seconds	Recorded response duration	—	—	INTEGER																					
	transcript_text	Transcript generated from the active recording where applicable	—	—	TEXT																					
	transcription_status	PENDING, PROCESSING, COMPLETED, or FAILED	—	—	VARCHAR(20)																					
	is_active	True only for the latest submitted version selected for analysis	—	—	BOOLEAN																					
	recorded_at	Recording timestamp	—	—	TIMESTAMPTZ																					
	submitted_at	Recording submission timestamp	—	—	TIMESTAMPTZ																					
	superseded_at	When a newer submitted recording replaced this active version	—	—	TIMESTAMPTZ																					
																										
																										
50	organization_owner_assignments																									
	Column	Purpose	PK	FK	Data Type																					
	owner_assignment_id	Ownership-assignment identifier	PK	—	UUID																					
	organization_id	Owned organisation	—	FK → organizations.organization_id	UUID																					
	owner_keycloak_user_id	Organization Owner identity	—	FK → users.keycloak_user_id	UUID																					
	onboarding_request_id	Onboarding request that established ownership	—	FK → organization_onboarding_requests.onboarding_request_id	UUID																					
	ownership_status	ACTIVE, REVOKED, or REPLACED	—	—	VARCHAR(30)																					
	assigned_at	Ownership activation time	—	—	TIMESTAMPTZ																					
	revoked_at	Ownership revocation time	—	—	TIMESTAMPTZ																					
	revocation_reason	Reason for revocation/replacement	—	—	TEXT																					
	created_at	Record creation time	—	—	TIMESTAMPTZ																					
																										
51	organization_admin_appointments																									
	Column	Purpose	PK	FK	Data Type																					
	appointment_id	Organization Admin appointment identifier	PK	—	UUID																					
	organization_id	Target organisation	—	FK → organizations.organization_id	UUID																					
	initiated_by_owner_assignment_id	Organization Owner initiating appointment/replacement	—	FK → organization_owner_assignments.owner_assignment_id	UUID																					
	outgoing_membership_id	Current Admin being replaced; NULL for first appointment	—	FK → organization_memberships.membership_id	UUID																					
	incoming_keycloak_user_id	Proposed incoming Organization Admin	—	FK → users.keycloak_user_id	UUID																					
	incoming_membership_id	Activated membership; NULL until activation	—	FK → organization_memberships.membership_id	UUID																					
	status	INVITED, EMAIL_VERIFICATION_PENDING, ACCEPTANCE_PENDING, ACTIVE, REJECTED, EXPIRED, ACTIVATION_FAILED, or CANCELLED	—	—	VARCHAR(40)																					
	invitation_token_reference	Opaque Keycloak invitation reference	—	—	VARCHAR(255)																					
	invitation_expires_at	Invitation expiry	—	—	TIMESTAMPTZ																					
	activated_at	Successful appointment time	—	—	TIMESTAMPTZ																					
	failure_reason	Recoverable activation failure reason	—	—	TEXT																					
	created_at	Request time	—	—	TIMESTAMPTZ																					
	updated_at	Last lifecycle update	—	—	TIMESTAMPTZ																					
																										
52	opening_version_reviews																									
	Column	Purpose	PK	FK	Data Type																					
	opening_review_id	Opening-package review identifier	PK	—	UUID																					
	opening_version_id	Reviewed opening version	—	FK → opening_versions.opening_version_id	UUID																					
	reviewed_by_membership_id	HR reviewer	—	FK → organization_memberships.membership_id	UUID																					
	review_status	APPROVED, RETURNED, or REJECTED	—	—	VARCHAR(30)																					
	comment	Required for returned/rejected packages	—	—	TEXT																					
	reviewed_at	Decision time	—	—	TIMESTAMPTZ																					
	created_at	Record creation time	—	—	TIMESTAMPTZ																					
																										
53	pre_screen_decision_batch_items																									
	Column	Purpose	PK	FK	Data Type																					
	decision_batch_item_id	Batch-item identifier	PK	—	UUID																					
	decision_batch_id	Parent controlled bulk decision	—	FK → pre_screen_decision_batches.decision_batch_id	UUID																					
	application_id	Explicitly selected application	—	FK → job_applications.application_id	UUID																					
	hr_decision_id	Individual decision created for this application	—	FK → hr_decisions.decision_id	UUID																					
	item_status	PENDING_VALIDATION, ELIGIBLE, DECIDED, SKIPPED, or FAILED	—	—	VARCHAR(30)																					
	validation_reason	Why an application was skipped or failed	—	—	TEXT																					
	created_at	Item creation time	—	—	TIMESTAMPTZ																					
	updated_at	Last item update	—	—	TIMESTAMPTZ																					
																										
54	requisition_version_stage_question_options																									
	Column	Purpose	PK	FK	Data Type																					
	requisition_version_option_id	Immutable requisition-question option identifier	PK	—	UUID																					
	requisition_version_question_id	Parent versioned question	—	FK → requisition_version_stage_questions.requisition_version_question_id	UUID																					
	option_text	Option displayed to applicant	—	—	TEXT																					
	option_order	Display order	—	—	SMALLINT																					
	is_correct	Correct option for auto-scored questions	—	—	BOOLEAN																					
	score_value	Marks for this option	—	—	NUMERIC(6,2)																					
	created_at	Creation time	—	—	TIMESTAMPTZ																					
																										
55	requisition_version_interview_criteria																									
	Column	Purpose	PK	FK	Data Type																					
	requisition_version_criteria_id	Immutable interview-criterion identifier	PK	—	UUID																					
	requisition_version_stage_id	Parent Live Interview stage	—	FK → requisition_version_stages.requisition_version_stage_id	UUID																					
	criteria_name	Criterion label	—	—	VARCHAR(100)																					
	weight	Criterion contribution; all criteria total 100	—	—	SMALLINT																					
	max_score	Highest score allowed	—	—	NUMERIC(6,2)																					
	guidance	Expected evidence and interviewer guidance	—	—	TEXT																					
	is_required	Whether a score is mandatory	—	—	BOOLEAN																					
	rating_scale	Approved labels/range	—	—	JSONB																					
	criteria_order	Display order	—	—	SMALLINT																					
	created_at	Creation time	—	—	TIMESTAMPTZ																					
																										
																										
																																																
																										

## Reconciliation decisions

Record any differences between the final list and the current `pente_db_diagram.dbml` here. This will be the source for the prototype's table catalogue, workflow-state mapping, and screen-to-data mapping.

| Decision | Final answer | Rationale / owner | Date |
|---|---|---|---|
| Database migrations available? | No | No migrations directory was supplied. | 2026-11-24 |
| Replacement schema | `mockup update/pente_db_diagram.dbml` | Replaces the previous design diagram for prototype work. | 2026-11-24 |
| Tenant-scoping convention | Direct or relationship-derived | Cross-table tenant/version checks are required in migrations. | 2026-11-24 |

