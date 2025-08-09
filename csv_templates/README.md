# Library Management CSV Templates

This folder contains sample CSV files for bulk data import into the Library Management System.

## Files Included

### 1. authors_master_data.csv
**Purpose**: Import author information
**Fields**: author_name, nationality, birth_year, death_year, biography, subject_specialization
**Records**: 15 sample authors

### 2. publishers_master_data.csv  
**Purpose**: Import publisher information
**Fields**: publisher_name, country, founded_year, website, specialization, contact_email
**Records**: 15 major publishers

### 3. library_members_bulk.csv
**Purpose**: Bulk member registration
**Fields**: member_id, first_name, last_name, email, phone, member_type, department, student_id, faculty_id, address, membership_plan
**Records**: 15 sample members (students, faculty, staff)

### 4. book_collection_catalog.csv
**Purpose**: Initial book catalog import
**Fields**: isbn, title, authors, publisher, publication_year, edition, pages, subject_code, price, copies_available, location, shelf_number
**Records**: 20 diverse book collection

## Usage Instructions

### 1. Data Import Process
1. Navigate to **Data → Data Import** in ERPNext
2. Select the appropriate DocType
3. Upload the CSV file
4. Map columns to ERPNext fields
5. Validate and import

### 2. Preparation Steps
- Ensure DocTypes exist before importing
- Verify field names match ERPNext schema
- Check data formats (dates, numbers, emails)
- Validate required fields are populated

### 3. Import Order
Recommended import sequence:
1. Authors Master Data
2. Publishers Master Data  
3. Subject Categories
4. Library Members
5. Book Collection Catalog
6. Circulation Data

### 4. Post-Import Validation
- Verify record counts
- Check for duplicate entries
- Test relationships between documents
- Generate validation reports

## Customization Notes

- Modify field names to match your DocType schema
- Adjust data formats based on regional requirements
- Add/remove fields as needed for your implementation
- Use proper encoding (UTF-8) for special characters

## Sample Data Details

All sample data is fictional and created for demonstration purposes. Real implementations should use actual institutional data following privacy and security guidelines.

For production use:
- Replace with actual member information
- Use real ISBN numbers for books
- Update publisher contact information
- Adjust pricing to current market rates
- Configure proper access controls