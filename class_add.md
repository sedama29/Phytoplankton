# Project and Class Operations Flowchart

This document provides a comprehensive flowchart covering all project and class management operations: Project Creation (with class addition), Class Addition, Class Modification, Class Merging, and Class Splitting.

---

## Complete Operations Flowchart

```mermaid
flowchart TB
    Start([User Accesses System]) --> MainMenu{Select Operation}
    
    %% PROJECT CREATION PATH
    MainMenu -->|Create/Update Project| ProjectFlow[Project Creation Flow]
    ProjectFlow --> CheckAuth{User<br/>Authenticated?}
    CheckAuth -->|No| AuthError[Show Error:<br/>Must be logged in]
    AuthError --> Start
    
    CheckAuth -->|Yes| ShowProjectForm[Display Project Form]
    ShowProjectForm --> UserProjectInput[User Enters:<br/>• project_code 3 chars<br/>• project_title<br/>• description<br/>• poc_name, poc_email, poc_phone<br/>• latitude, longitude<br/>• area, dates, privacy<br/>• Optional: classes array]
    
    UserProjectInput --> ValidateProject{Validate<br/>Project Data}
    ValidateProject -->|Invalid| ShowValidationError[Show Validation Error]
    ShowValidationError --> UserProjectInput
    
    ValidateProject -->|Valid| CheckProjectExists{Project Code<br/>Already Exists?}
    CheckProjectExists -->|Yes| UpdateProject[POST /submit<br/>Update Project]
    CheckProjectExists -->|No| CreateProject[POST /submit<br/>Create Project]
    
    UpdateProject --> UpdateProjectDB[(Database:<br/>UPDATE projects)]
    UpdateProjectDB --> ProjectUpdated[Project Updated]
    
    CreateProject --> InsertProjectDB[(Database:<br/>INSERT INTO projects)]
    InsertProjectDB --> CheckClassesProvided{Classes<br/>Provided?}
    
    CheckClassesProvided -->|No| ProjectCreated[Project Created<br/>No Classes]
    CheckClassesProvided -->|Yes| ProcessClasses[Process Classes Array]
    
    ProcessClasses --> LoopClasses{For Each Class}
    LoopClasses --> ValidateClassFormat{Validate<br/>Class Name Format}
    ValidateClassFormat -->|Invalid| ClassFormatError[Error:<br/>lowercase_underscores only]
    ClassFormatError --> LoopClasses
    
    ValidateClassFormat -->|Valid| CheckModelClass{Class in Model<br/>Training Set?}
    CheckModelClass -->|Yes| SetModelFlag[Set is_model_class = 1]
    CheckModelClass -->|No| SetManualFlag[Set is_model_class = 0<br/>Show Warning]
    
    SetModelFlag --> InsertClassDB[(INSERT INTO class_details<br/>project_code, class_name,<br/>formatted_class_name,<br/>description, is_model_class)]
    SetManualFlag --> InsertClassDB
    
    InsertClassDB --> CheckMoreClasses{More Classes?}
    CheckMoreClasses -->|Yes| LoopClasses
    CheckMoreClasses -->|No| CommitProject[COMMIT Transaction]
    
    CommitProject --> ProjectCreated
    ProjectCreated --> ShowProjectSuccess[Show Success Message]
    ProjectUpdated --> ShowProjectSuccess
    ShowProjectSuccess --> EndProject([Project Operation Complete])
    
    %% CLASS ADDITION PATH
    MainMenu -->|Add Class| ClassAddFlow[Class Addition Flow]
    ClassAddFlow --> CheckProjectSelected{Project<br/>Selected?}
    CheckProjectSelected -->|No| ProjectError[Show Error:<br/>Select Project First]
    ProjectError --> ClassAddFlow
    
    CheckProjectSelected -->|Yes| ShowClassForm[Display Class<br/>Creation Form]
    ShowClassForm --> UserClassInput[User Enters:<br/>• class_name lowercase_underscores<br/>• formatted_class_name<br/>• description optional]
    
    UserClassInput --> ValidateClassInput{Validate Input}
    ValidateClassInput -->|Empty Name| NameError[Show Error:<br/>Class name required]
    NameError --> UserClassInput
    
    ValidateClassInput -->|Invalid Format| FormatError[Show Error:<br/>Must be lowercase_underscores]
    FormatError --> UserClassInput
    
    ValidateClassInput -->|Valid| CheckModelClassAdd{Class in Model<br/>Training Set?}
    CheckModelClassAdd -->|No| ShowModelWarning[Show Warning:<br/>Class not in model.<br/>Will require manual classification]
    ShowModelWarning --> UserConfirm{User<br/>Confirms?}
    UserConfirm -->|No| UserClassInput
    UserConfirm -->|Yes| CreateClassAPI[POST /create_class]
    
    CheckModelClassAdd -->|Yes| CreateClassAPI
    
    CreateClassAPI --> BackendValidate{Backend<br/>Validation}
    BackendValidate -->|Project Invalid| ReturnProjectErr[Return Error:<br/>Project does not exist]
    ReturnProjectErr --> ProjectError
    
    BackendValidate -->|Class Exists| ReturnClassErr[Return Error:<br/>Class already exists<br/>for this project]
    ReturnClassErr --> UserClassInput
    
    BackendValidate -->|Valid| InsertClassDetails[(Database:<br/>INSERT INTO class_details<br/>project_code, class_name,<br/>formatted_class_name,<br/>description, is_model_class)]
    
    InsertClassDetails --> CheckUniqueConstraint{Unique Constraint<br/>project_code + class_name?}
    CheckUniqueConstraint -->|Violation| ReturnClassErr
    CheckUniqueConstraint -->|OK| CommitClass[COMMIT Transaction]
    
    CommitClass --> ReturnClassSuccess[Return Success Response<br/>with is_model_class flag]
    ReturnClassSuccess --> UpdateClassUI[Update Frontend UI]
    UpdateClassUI --> RefreshClassList[Refresh Class List<br/>for Project]
    RefreshClassList --> ShowClassSuccess[Show Success Message]
    ShowClassSuccess --> EndClassAdd([Class Added Successfully])
    
    %% CLASS MODIFICATION PATH
    MainMenu -->|Modify Class| ClassModifyFlow[Class Modification Flow]
    ClassModifyFlow --> CheckProjectModify{Project<br/>Selected?}
    CheckProjectModify -->|No| ModifyProjectError[Show Error:<br/>Select Project First]
    ModifyProjectError --> ClassModifyFlow
    
    CheckProjectModify -->|Yes| LoadClassList[Load Classes<br/>for Project]
    LoadClassList --> ShowClassList[Display Class List<br/>with Edit Options]
    ShowClassList --> UserSelectClass[User Selects Class<br/>to Modify]
    
    UserSelectClass --> LoadClassDetails[Load Class Details<br/>from Database]
    LoadClassDetails --> ShowEditForm[Display Edit Form<br/>with Current Values]
    ShowEditForm --> UserModifyInput[User Modifies:<br/>• formatted_class_name<br/>• description<br/>Note: class_name cannot change]
    
    UserModifyInput --> ValidateModify{Validate<br/>Input?}
    ValidateModify -->|Invalid| ModifyValidationError[Show Validation Error]
    ModifyValidationError --> UserModifyInput
    
    ValidateModify -->|Valid| CheckAuthModify{User<br/>Authenticated?}
    CheckAuthModify -->|No| ModifyAuthError[Show Error:<br/>Must be logged in]
    ModifyAuthError --> Start
    
    CheckAuthModify -->|Yes| UpdateClassAPI[POST /submit_class_details]
    UpdateClassAPI --> CheckClassExists{Class<br/>Exists?}
    CheckClassExists -->|No| ClassNotExists[Return Error:<br/>Class does not exist]
    ClassNotExists --> UserSelectClass
    
    CheckClassExists -->|Yes| UpdateClassDB[(Database:<br/>UPDATE class_details<br/>SET formatted_class_name = ?,<br/>description = ?<br/>WHERE class_name = ?)]
    
    UpdateClassDB --> CommitModify[COMMIT Transaction]
    CommitModify --> ReturnModifySuccess[Return Success Response]
    ReturnModifySuccess --> UpdateModifyUI[Update Frontend UI]
    UpdateModifyUI --> RefreshModifyList[Refresh Class List]
    RefreshModifyList --> ShowModifySuccess[Show Success Message]
    ShowModifySuccess --> EndClassModify([Class Modified Successfully])
    
    %% CLASS MERGING PATH
    MainMenu -->|Merge Classes| MergeFlow[Class Merging Flow]
    MergeFlow --> CheckProjectMerge{Project<br/>Selected?}
    CheckProjectMerge -->|No| MergeProjectError[Show Error:<br/>Select Project First]
    MergeProjectError --> MergeFlow
    
    CheckProjectMerge -->|Yes| LoadProjectClasses[Load Classes<br/>for Project]
    LoadProjectClasses --> ShowClassListMerge[Display Class List<br/>with Multi-Select]
    ShowClassListMerge --> UserSelectSources[User Selects Source Classes<br/>2 or more classes to merge]
    
    UserSelectSources --> CheckSelection{At Least 2<br/>Classes Selected?}
    CheckSelection -->|No| SelectionError[Show Error:<br/>Select at least 2 classes]
    SelectionError --> UserSelectSources
    
    CheckSelection -->|Yes| UserSelectTarget[User Selects Target Class<br/>Class to merge into]
    UserSelectTarget --> ValidateTarget{Target Class<br/>Selected?}
    ValidateTarget -->|No| TargetError[Show Error:<br/>Select target class]
    TargetError --> UserSelectTarget
    
    ValidateTarget -->|Yes| CheckSameProject{All Classes<br/>Same Project?}
    CheckSameProject -->|No| ProjectMismatch[Show Error:<br/>Cannot merge classes<br/>from different projects]
    ProjectMismatch --> UserSelectSources
    
    CheckSameProject -->|Yes| CheckTargetExists{Target Class<br/>Exists?}
    CheckTargetExists -->|No| TargetNotExists[Show Error:<br/>Target class does not exist]
    TargetNotExists --> UserSelectTarget
    
    CheckTargetExists -->|Yes| CheckTargetInSources{Target in<br/>Source List?}
    CheckTargetInSources -->|Yes| TargetInSources[Show Error:<br/>Target cannot be<br/>in source list]
    TargetInSources --> UserSelectTarget
    
    CheckTargetInSources -->|No| GetImageCounts[Get Image Counts<br/>for Each Source Class]
    GetImageCounts --> ShowMergePreview[Show Preview:<br/>• Source classes list<br/>• Target class<br/>• Image count per class<br/>• Total images to move]
    
    ShowMergePreview --> UserConfirmMerge{User Confirms<br/>Merge?}
    UserConfirmMerge -->|No| CancelMerge[Cancel Operation]
    CancelMerge --> EndMerge
    
    UserConfirmMerge -->|Yes| StartMergeTransaction[Start Database Transaction<br/>BEGIN TRANSACTION]
    
    StartMergeTransaction --> LoopMergeClasses{For Each Source Class}
    LoopMergeClasses --> UpdateImageData[(UPDATE image_data<br/>SET class_name = target_class<br/>WHERE class_name = source_class<br/>AND bin_name IN<br/>SELECT bin_name FROM bin_data<br/>WHERE project_code = ?)]
    
    UpdateImageData --> GetYearMonth[Extract year/month<br/>from bin_name pattern]
    GetYearMonth --> LoopFolders{For Each Year/Month<br/>Combination}
    
    LoopFolders --> GetSourcePath[Get Source Path:<br/>data_directory/project/year/month/source_class/]
    GetSourcePath --> GetTargetPath[Get Target Path:<br/>data_directory/project/year/month/target_class/]
    
    GetTargetPath --> CheckTargetFolder{Target Folder<br/>Exists?}
    CheckTargetFolder -->|No| CreateTargetFolder[Create Target Folder<br/>os.makedirs target_path]
    CheckTargetFolder -->|Yes| MoveImageFiles
    
    CreateTargetFolder --> MoveImageFiles[Move Image Files<br/>shutil.move all files<br/>from source to target]
    
    MoveImageFiles --> CheckEmptySource{Source Folder<br/>Empty?}
    CheckEmptySource -->|Yes| RemoveSourceFolder[Remove Empty Folder<br/>os.rmdir source_path]
    CheckEmptySource -->|No| CheckMoreFolders
    
    RemoveSourceFolder --> CheckMoreFolders{More Year/Month<br/>Combinations?}
    CheckMoreFolders -->|Yes| LoopFolders
    CheckMoreFolders -->|No| CheckMoreSourceClasses{More Source<br/>Classes?}
    
    CheckMoreSourceClasses -->|Yes| LoopMergeClasses
    CheckMoreSourceClasses -->|No| DeleteSourceClasses[(DELETE FROM class_details<br/>WHERE class_name IN source_classes<br/>AND project_code = ?)]
    
    DeleteSourceClasses --> CommitMergeTransaction[COMMIT Transaction]
    CommitMergeTransaction --> LogMergeOperation[(Log to audit table<br/>operation_type = 'merge'<br/>source_classes, target_class<br/>images_affected)]
    
    LogMergeOperation --> ReturnMergeSuccess[Return Success Response<br/>with merge summary]
    ReturnMergeSuccess --> UpdateMergeUI[Update Frontend UI]
    UpdateMergeUI --> RefreshMergeClassList[Refresh Class List]
    RefreshMergeClassList --> RefreshMergeImages[Refresh Image List]
    RefreshMergeImages --> ShowMergeSuccess[Show Success Message<br/>with merge summary]
    ShowMergeSuccess --> EndMerge([Merge Complete])
    
    %% CLASS SPLITTING PATH
    MainMenu -->|Split Class| SplitFlow[Class Splitting Flow]
    SplitFlow --> CheckProjectSplit{Project<br/>Selected?}
    CheckProjectSplit -->|No| SplitProjectError[Show Error:<br/>Select Project First]
    SplitProjectError --> SplitFlow
    
    CheckProjectSplit -->|Yes| SelectSourceClass[User Selects Source Class<br/>Class to split from]
    SelectSourceClass --> ValidateSource{Source Class<br/>Exists?}
    ValidateSource -->|No| SourceError[Show Error:<br/>Class does not exist]
    SourceError --> SelectSourceClass
    
    ValidateSource -->|Yes| LoadSourceImages[Load Images<br/>from Source Class]
    LoadSourceImages --> ShowImageGrid[Display Image Grid<br/>with Selection UI<br/>Checkboxes or Drag Selection]
    
    ShowImageGrid --> UserSelectImages[User Selects Images<br/>to Split Out]
    UserSelectImages --> CheckImagesSelected{At Least 1<br/>Image Selected?}
    CheckImagesSelected -->|No| NoImagesError[Show Error:<br/>Select at least 1 image]
    NoImagesError --> UserSelectImages
    
    CheckImagesSelected -->|Yes| CreateTargetClass[User Creates/Selects<br/>Target Class Name]
    CreateTargetClass --> ValidateTargetName{Target Class Name<br/>Valid Format?}
    ValidateTargetName -->|No| TargetNameError[Show Error:<br/>Invalid class name format]
    TargetNameError --> CreateTargetClass
    
    ValidateTargetName -->|Yes| CheckTargetExistsSplit{Target Class<br/>Exists?}
    CheckTargetExistsSplit -->|No| CreateNewClassSplit[Create New Class<br/>INSERT INTO class_details]
    CheckTargetExistsSplit -->|Yes| ValidateTargetProject{Target Class<br/>Same Project?}
    
    CreateNewClassSplit --> ValidateTargetProject
    ValidateTargetProject -->|No| SplitProjectMismatch[Show Error:<br/>Target class must be<br/>in same project]
    SplitProjectMismatch --> CreateTargetClass
    
    ValidateTargetProject -->|Yes| GetSelectedImageNames[Extract Image Names<br/>from Selected Images]
    GetSelectedImageNames --> ShowSplitPreview[Show Preview:<br/>• Source class<br/>• Target class<br/>• Selected image count<br/>• Remaining image count<br/>• Option: Delete source if empty]
    
    ShowSplitPreview --> UserConfirmSplit{User Confirms<br/>Split?}
    UserConfirmSplit -->|No| CancelSplit[Cancel Operation]
    CancelSplit --> EndSplit
    
    UserConfirmSplit -->|Yes| StartSplitTransaction[Start Database Transaction<br/>BEGIN TRANSACTION]
    
    StartSplitTransaction --> LoopSplitImages{For Each Selected Image}
    LoopSplitImages --> GetImageBinName[Get bin_name for Image<br/>from image_data]
    
    GetImageBinName --> ExtractYearMonth[Extract year/month<br/>from bin_name pattern<br/>IFCBXXX_YYYYMMDD...]
    ExtractYearMonth --> GetSourcePathSplit[Source Path:<br/>project/year/month/source_class/image.png]
    
    GetSourcePathSplit --> GetTargetPathSplit[Target Path:<br/>project/year/month/target_class/image.png]
    
    GetTargetPathSplit --> CheckTargetFolderSplit{Target Folder<br/>Exists?}
    CheckTargetFolderSplit -->|No| CreateTargetFolderSplit[Create Target Folder<br/>os.makedirs]
    CheckTargetFolderSplit -->|Yes| UpdateImageRecord
    
    CreateTargetFolderSplit --> UpdateImageRecord[(UPDATE image_data<br/>SET class_name = target_class<br/>WHERE file_name = image_name<br/>AND class_name = source_class<br/>AND bin_name IN<br/>SELECT bin_name FROM bin_data<br/>WHERE project_code = ?)]
    
    UpdateImageRecord --> MovePhysicalFile[Move Physical File<br/>shutil.move source_path<br/>to target_path]
    
    MovePhysicalFile --> CheckMoreSplitImages{More Images<br/>to Process?}
    CheckMoreSplitImages -->|Yes| LoopSplitImages
    CheckMoreSplitImages -->|No| CheckSourceEmpty{Source Class<br/>Empty Now?}
    
    CheckSourceEmpty -->|Yes| CheckDeleteOption{User Selected<br/>Delete If Empty?}
    CheckDeleteOption -->|Yes| DeleteSourceClass[(DELETE FROM class_details<br/>WHERE project_code = ?<br/>AND class_name = ?)]
    CheckDeleteOption -->|No| KeepSourceClass[Keep Source Class<br/>with remaining images]
    
    CheckSourceEmpty -->|No| KeepSourceClass
    DeleteSourceClass --> CommitSplitTransaction[COMMIT Transaction]
    KeepSourceClass --> CommitSplitTransaction
    
    CommitSplitTransaction --> LogSplitOperation[(Log to audit table<br/>operation_type = 'split'<br/>source_class, target_class<br/>images_affected)]
    
    LogSplitOperation --> ReturnSplitSuccess[Return Success Response<br/>with split summary]
    ReturnSplitSuccess --> UpdateSplitUI[Update Frontend UI]
    UpdateSplitUI --> RefreshSplitClassList[Refresh Class List]
    RefreshSplitClassList --> RefreshSplitImages[Refresh Image Lists<br/>for Both Classes]
    RefreshSplitImages --> ShowSplitSuccess[Show Success Message<br/>with split summary]
    ShowSplitSuccess --> EndSplit([Split Complete])
    
    %% END POINTS
    EndProject --> Start
    EndClassAdd --> Start
    EndClassModify --> Start
    EndMerge --> Start
    EndSplit --> Start
    
    %% STYLING
    style Start fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    style EndProject fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style EndClassAdd fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style EndClassModify fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style EndMerge fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style EndSplit fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    
    style InsertProjectDB fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style InsertClassDB fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style InsertClassDetails fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style UpdateClassDB fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style UpdateImageData fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style DeleteSourceClasses fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style UpdateImageRecord fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style DeleteSourceClass fill:#fff3e0,stroke:#e65100,stroke-width:2px
    
    style AuthError fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style ShowValidationError fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style ProjectError fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style NameError fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style FormatError fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style ReturnClassErr fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style ModifyProjectError fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style ModifyValidationError fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style ModifyAuthError fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style ClassNotExists fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style SelectionError fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style TargetError fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style ProjectMismatch fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style TargetNotExists fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style TargetInSources fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style SourceError fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style NoImagesError fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style TargetNameError fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style SplitProjectMismatch fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    
    style ShowModelWarning fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    style ShowMergePreview fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    style ShowSplitPreview fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    
    style StartMergeTransaction fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    style StartSplitTransaction fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    style CommitProject fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    style CommitClass fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    style CommitModify fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    style CommitMergeTransaction fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    style CommitSplitTransaction fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
```

---

## Operation Summaries

### 1. Project Creation (with Class Addition)

**Flow**: User Input → Validation → Check Existence → Create/Update → Process Classes (if provided) → Commit

**Key Steps**:
- User authentication required
- Validate all project fields (code, title, email, phone, coordinates, etc.)
- Check if project code already exists (update vs. create)
- If classes provided: validate each class, check if in model training set, set `is_model_class` flag
- All operations in a single transaction

**API Endpoint**: `POST /submit`

**Database Operations**:
- `INSERT INTO projects` or `UPDATE projects`
- `INSERT INTO class_details` (for each class if provided)

---

### 2. Class Addition

**Flow**: Select Project → Enter Class Details → Validate → Check Model → Create → Update UI

**Key Steps**:
- Project must be selected first
- Validate class name format (lowercase_underscores)
- Check if class is in model's training set
- Show warning if class not in model (requires manual classification)
- Create class in database with `is_model_class` flag
- Refresh class list in UI

**API Endpoint**: `POST /create_class`

**Database Operations**:
- `INSERT INTO class_details` (with unique constraint on project_code + class_name)

---

### 3. Class Modification

**Flow**: Select Project → Select Class → Load Details → Edit → Validate → Update → Refresh UI

**Key Steps**:
- Project must be selected first
- Load existing class details from database
- User can modify `formatted_class_name` and `description`
- Class name (`class_name`) cannot be changed
- User authentication required
- Update database and refresh UI

**API Endpoint**: `POST /submit_class_details`

**Database Operations**:
- `UPDATE class_details SET formatted_class_name = ?, description = ? WHERE class_name = ?`

---

### 4. Class Merging

**Flow**: Select Project → Select Sources (2+) → Select Target → Validate → Preview → Merge → Move Files → Update DB

**Key Steps**:
- Project must be selected first
- Select 2 or more source classes to merge
- Select target class (must exist, cannot be in source list)
- All classes must belong to same project
- Show preview with image counts
- Start transaction
- Update all image_data records
- Move physical files from source to target folders
- Delete source classes from class_details
- Log operation to audit table

**API Endpoint**: `POST /merge_classes`

**Database Operations**:
- `UPDATE image_data SET class_name = target WHERE class_name = source`
- `DELETE FROM class_details WHERE class_name IN source_classes`
- `INSERT INTO class_operations_audit`

**File Operations**:
- Move files: `project/year/month/source_class/` → `project/year/month/target_class/`
- Remove empty source folders

---

### 5. Class Splitting

**Flow**: Select Project → Select Source → Select Images → Create/Select Target → Preview → Split → Move Files → Update DB

**Key Steps**:
- Project must be selected first
- Select source class to split from
- Load and display images from source class
- User selects images to split out
- Create or select target class (must be same project)
- Show preview with counts and option to delete source if empty
- Start transaction
- For each selected image: update database and move file
- Optionally delete source class if empty
- Log operation to audit table

**API Endpoint**: `POST /split_class`

**Database Operations**:
- `UPDATE image_data SET class_name = target WHERE file_name = image AND class_name = source`
- `INSERT INTO class_details` (if creating new target class)
- `DELETE FROM class_details` (if source class becomes empty and user selected delete option)
- `INSERT INTO class_operations_audit`

**File Operations**:
- Move individual files: `project/year/month/source_class/image.png` → `project/year/month/target_class/image.png`
- Create target folder if needed

---

## Common Validation Rules

### Project Validation
- Project code: 3 alphanumeric characters
- Email: Valid email format
- Phone: Valid phone format (if provided)
- Coordinates: Valid latitude/longitude values
- Title: Cannot contain URLs

### Class Validation
- Class name: lowercase with underscores only (e.g., `class_name`)
- Class name must be unique per project
- Formatted class name: Required, can contain spaces and special characters
- Description: Optional

### Model Class Check
- System checks if class exists in model's training set
- If yes: `is_model_class = 1` (can be auto-classified)
- If no: `is_model_class = 0` (requires manual classification, shows warning)

---

## Transaction Management

All operations that modify multiple database records or files use transactions:

1. **Project Creation with Classes**: Project + all classes in one transaction
2. **Class Merging**: Multiple image updates + file moves + class deletions in one transaction
3. **Class Splitting**: Multiple image updates + file moves + optional class deletion in one transaction

Transactions ensure atomicity: either all changes succeed or all are rolled back.

---

## Error Handling

All operations include comprehensive error handling:
- **Validation Errors**: Returned immediately with specific error messages
- **Database Errors**: Caught and returned as error responses
- **File System Errors**: Caught during file operations, transaction rolled back
- **Constraint Violations**: Unique constraint violations caught and reported

---

## Audit Logging

Class merging and splitting operations are logged to `class_operations_audit` table:
- Operation type ('merge' or 'split')
- Project code
- Source classes (JSON array)
- Target class
- Number of images affected
- User who performed the operation
- Timestamp

---

This flowchart provides a complete visual guide to all project and class management operations in the plankton classifier system.

