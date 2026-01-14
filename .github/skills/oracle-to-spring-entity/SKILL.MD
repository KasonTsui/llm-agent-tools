---
name: oracle-to-spring-entity
description: Converts Oracle SQL DDL into production-ready Spring Boot JPA Entities and Repositories. Use this when the user provides a SQL CREATE TABLE statement or asks to generate an entity from a database table.
---

# Oracle SQL to Spring Data JPA Conversion Skill

When the user provides an Oracle SQL DDL statement (CREATE TABLE) or asks to generate a Java Entity, follow this strict procedure to generate production-ready Spring Boot 3+ code.

## 1. Analysis Phase
Analyze the input DDL for:
- **Table Name & Schema:** Identify the schema (e.g., `"ABC"`) and table name.
- **Primary Key:** Identify the PK column.
- **ID Strategy:** Check for `GENERATED AS IDENTITY`. If missing, assume an Oracle SEQUENCE is used.
- **Booleans:** Identify `NUMBER(1,0)` or `NUMBER(1)` columns (standard Oracle boolean pattern).
- **Audit Columns:** Identify `CREATE_DATE`, `MODIFY_DATE`, or similar.
- **Constraints:** Identify `NOT NULL`, `UNIQUE`, and foreign keys.

## 2. Generation Rules (Strict)

### Class Structure
- Use Lombok annotations: `@Getter`, `@Setter`, `@NoArgsConstructor`, `@AllArgsConstructor`, `@Builder`.
- Use JPA annotations: `@Entity`, `@Table(name = "...", schema = "...")`, `@EntityListeners(AuditingEntityListener.class)`.
- **Naming:** Convert `SNAKE_CASE` DB names to `PascalCase` classes and `camelCase` fields.

### Field Mapping
| Oracle Type | Java Type | Annotation / Converter |
| :--- | :--- | :--- |
| `NUMBER(1)` or `NUMBER(1,0)` | `Boolean` | `@Convert(converter = NumericBooleanConverter.class)` |
| `DATE` / `TIMESTAMP` | `Instant` | `@Column(...)` (Use `Instant` for safer time handling) |
| `VARCHAR2(N)` | `String` | `@Column(length = N)` |
| `NUMBER(P, 0)` (P > 1) | `Long` | For IDs or Large Integers |
| `NUMBER(P, S)` (S > 0) | `BigDecimal` | For currency/precision |

### Special Handling
1.  **ID Generation:**
    - If `GENERATED AS IDENTITY` is present: Use `@GeneratedValue(strategy = GenerationType.IDENTITY)`.
    - If NOT present: Comment out a `@SequenceGenerator` block so the user can fill in the sequence name.
2.  **Audit Fields:**
    - Map creation dates to `@CreatedDate`.
    - Map modification dates to `@LastModifiedDate`.
    - Add `updatable = false` to `@CreatedDate` column.
3.  **Validation:**
    - Add `@NotNull` for `NOT NULL` columns (except ID/Audit fields which are auto-managed).
    - Add `@Size(max = N)` for Strings.

## 3. Repository Generation
- Create an interface extending `JpaRepository<EntityName, IdType>`.
- Add `Optional<EntityName> findByUniqueColumn(...)` for any unique constraints found in the DDL.

## 4. Final Output Format
Provide the response in this structure:

### 1. Entity: `[ClassName].java`

    // Complete Java Code

### 2. Repository: `[ClassName]Repository.java`

    // Complete Interface Code

### 3. ⚠️ Migration Checklist
- [ ] **Sequence Name:** Verified sequence name matches DB (if using sequences).
- [ ] **Boolean Logic:** Confirmed `NUMBER(1)` is meant to be Boolean (0/1).
- [ ] **Schema Permissions:** Confirmed app user has access to schema `[SCHEMA_NAME]`.

## Example
**Input:** `CREATE TABLE "APP"."USERS" ("IS_ACTIVE" NUMBER(1,0))`
**Output Field:**
    @Column(name = "IS_ACTIVE", precision = 1)
    @Convert(converter = NumericBooleanConverter.class)
    private Boolean isActive;
