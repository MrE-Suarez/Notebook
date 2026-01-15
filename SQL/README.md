# SQL Course Notes v2.0

Welcome to my personal SQL learning notes (Version 2.0).  
This repository follows the complete course structure, divided into three main sections: **Basics**, **Intermediate**, and **Advanced**.

---

## 📚 What's New in v2.0

### ✨ Improvements
- 🎨 **Better formatting** with consistent structure and clear hierarchy
- 📊 **Enhanced tables** for quick reference and comparisons
- 💡 **Practical examples** with real-world scenarios
- ⚡ **Quick reference sections** at the start of each document
- 🔗 **Internal navigation** with clear section links
- 📝 **Expanded explanations** for complex topics (Window Functions, Indexes, CTEs)
- 🗑️ **Removed clutter** (personal notes, incomplete sections)

### 🎯 Focus Areas
- **Readability**: Easy to scan and find specific topics
- **Completeness**: All sections properly documented
- **Professionalism**: Portfolio-ready format
- **Practicality**: Real-world use cases and best practices

---

## 📘 Course Overview

| Level | Topics Covered | File |
|-------|----------------|------|
| **Basics** | 1. Introduction<br>2. Query Data (SELECT)<br>3. Data Definition (DDL)<br>4. Data Manipulation (DML) | [BASICS_01-04_v2.md](./BASICS_01-04_v2.md) |
| **Intermediate** | 5. Filtering Data<br>6. Combining Data<br>7. Row-Level Functions<br>8. Aggregation & Analytical Functions | [INTERMEDIATE_05-08_v2.md](./INTERMEDIATE_05-08_v2.md) |
| **Advanced** | 9. Advanced SQL Techniques<br>10. Performance Optimization<br>11. AI & SQL<br>12. SQL Projects | [ADVANCED_09-12_v2_Part1.md](./ADVANCED_09-12_v2_Part1.md)<br>[ADVANCED_09-12_v2_Part2.md](./ADVANCED_09-12_v2_Part2.md) |

---

## 🧭 Navigation Guide

### 📘 BASICS (Sections 01–04)
**Foundation concepts for SQL syntax and database operations**

**Topics:**
- SELECT queries and data retrieval
- DDL: CREATE, ALTER, DROP tables
- DML: INSERT, UPDATE, DELETE, TRUNCATE
- Common clauses: WHERE, GROUP BY, HAVING, ORDER BY

**Best for:** Complete beginners, syntax review, interview prep basics

---

### 📗 INTERMEDIATE (Sections 05–08)
**Core SQL skills for data analysis and manipulation**

**Topics:**
- **Filtering**: Operators, pattern matching, logical conditions
- **Joins**: INNER, LEFT, RIGHT, FULL, Anti-joins, CROSS JOIN
- **Set Operators**: UNION, INTERSECT, EXCEPT
- **Functions**: String, number, date/time, NULL handling, CASE
- **Window Functions**: Aggregate, ranking, value functions

**Best for:** Data analysts, preparing for analytics roles, mastering window functions

---

### 📕 ADVANCED (Sections 09–12)

#### Part 1: Advanced Techniques
**Complex query patterns and database architecture**

**Topics:**
- Subqueries (scalar, row, table, correlated)
- CTEs (standalone, nested, recursive)
- Views vs Tables vs CTEs
- Temp tables and CTAS
- Stored procedures and triggers (overview)

**Best for:** Database developers, data engineers, complex analytics

#### Part 2: Performance Optimization
**Making queries faster and databases more efficient**

**Topics:**
- **Indexes**: Clustered, non-clustered, columnstore, composite
- **Index Management**: Monitoring, statistics, fragmentation
- **Execution Plans**: Reading and optimizing plans
- **Partitioning**: Table partitioning strategies
- **30 Performance Tips**: Complete optimization checklist
- **AI Integration**: ChatGPT prompts for SQL development

**Best for:** Performance tuning, production databases, senior roles

---

## 🎓 Skill Progression

```
BASICS → Learn syntax and fundamentals
   ↓
INTERMEDIATE → Master analytical queries
   ↓
ADVANCED (Part 1) → Understand complex patterns
   ↓
ADVANCED (Part 2) → Optimize for performance
```

---

## 🔍 Quick Topic Finder

### By Skill Level

**Beginner:**
- SELECT, WHERE, ORDER BY → BASICS
- Comparison operators → INTERMEDIATE (5.1)
- Basic JOINs → INTERMEDIATE (6.1)

**Intermediate:**
- Window functions → INTERMEDIATE (8.2-8.5)
- NULL handling → INTERMEDIATE (7.4)
- CASE statements → INTERMEDIATE (7.5)

**Advanced:**
- CTEs and subqueries → ADVANCED Part 1 (9.2-9.3)
- Index strategies → ADVANCED Part 2 (10.1-10.2)
- Performance optimization → ADVANCED Part 2 (10.6)

---

### By Use Case

**Data Analysis:**
- Window functions → INTERMEDIATE (8.2)
- Aggregations → INTERMEDIATE (8.1)
- Subqueries → ADVANCED Part 1 (9.2)

**Performance Tuning:**
- Indexes → ADVANCED Part 2 (10.1)
- Execution plans → ADVANCED Part 2 (10.3)
- 30 optimization tips → ADVANCED Part 2 (10.6)

**Database Design:**
- DDL best practices → BASICS (3)
- Views and CTEs → ADVANCED Part 1 (9.3-9.4)
- Partitioning → ADVANCED Part 2 (10.5)

**Interview Prep:**
- All BASICS topics
- JOINs → INTERMEDIATE (6.1)
- Window functions → INTERMEDIATE (8.2-8.5)
- Indexes → ADVANCED Part 2 (10.1)

---

## 💡 How to Use These Notes

### For Learning
1. **Start with BASICS** if you're new to SQL
2. **Progress to INTERMEDIATE** once comfortable with syntax
3. **Dive into ADVANCED** when ready for complex patterns
4. **Practice regularly** with the examples provided
5. **Use Quick Reference tables** for syntax reminders

### For Review
- Use the **Quick Reference** sections at the start of each document
- Navigate directly to specific topics using the table of contents
- Check **comparison tables** for choosing between similar concepts

### For Interviews
- Review **BASICS** for syntax questions
- Master **INTERMEDIATE** sections on JOINs and Window Functions
- Study **ADVANCED Part 2** performance tips
- Practice with **ChatGPT prompts** in ADVANCED (11.3)

### For Work
- Keep as a **syntax reference** on a second monitor
- Use **performance tips** when queries run slow
- Reference **index strategies** when designing tables
- Apply **best practices** from each section

---

## 🧰 Tools & Technologies

- **Primary DBMS:** Microsoft SQL Server
- **Editor:** SQL Server Management Studio (SSMS)
- **Version Control:** Git & GitHub
- **Documentation:** Markdown in VS Code
- **AI Assist:** ChatGPT, GitHub Copilot

---

## 📖 Additional Resources

### Recommended Practice
- **LeetCode SQL** → Interview-style problems
- **HackerRank SQL** → Structured learning path
- **DataLemur** → Real-world data analytics questions
- **SQLZoo** → Interactive tutorials

### Further Reading
- SQL Performance Explained by Markus Winand
- SQL Antipatterns by Bill Karwin
- Microsoft SQL Server Documentation

---

## 🎯 Learning Objectives

### By Section

**BASICS**
- ✅ Write SELECT queries with filtering and sorting
- ✅ Create and modify table structures
- ✅ Insert, update, and delete data safely
- ✅ Understand query execution order

**INTERMEDIATE**
- ✅ Join multiple tables effectively
- ✅ Use window functions for analytics
- ✅ Handle NULL values and dates
- ✅ Write conditional logic with CASE

**ADVANCED**
- ✅ Break complex queries into CTEs
- ✅ Design efficient indexes
- ✅ Read and optimize execution plans
- ✅ Apply 30 performance best practices
- ✅ Use AI tools for SQL development

---

## 📊 Progress Tracking

### Self-Assessment Checklist

**BASICS ✓**
- [ ] Can write basic SELECT queries
- [ ] Can create and modify tables
- [ ] Understand INSERT, UPDATE, DELETE
- [ ] Know when to use WHERE vs HAVING

**INTERMEDIATE ✓**
- [ ] Can join 3+ tables correctly
- [ ] Understand LEFT vs INNER JOIN
- [ ] Can use window functions
- [ ] Handle NULLs and dates confidently

**ADVANCED ✓**
- [ ] Can write recursive CTEs
- [ ] Understand index types
- [ ] Can read execution plans
- [ ] Apply performance optimization

---

## 🚀 Next Steps

### Current Status
- ✅ BASICS completed
- ✅ INTERMEDIATE completed
- ✅ ADVANCED techniques completed
- ✅ Performance optimization documented
- ⏳ SQL Projects (in progress)

### Future Additions
- [ ] Real-world project examples
- [ ] Practice exercise solutions
- [ ] Interview question bank
- [ ] Performance tuning case studies

---

## 📝 Notes Purpose

These notes serve as:
- 📚 **Personal study reference** for SQL mastery
- 💼 **Portfolio piece** showcasing documentation skills
- 🔄 **Version-controlled learning** tracking progress over time
- 🌐 **Public resource** for others learning SQL

---

## 🤝 Contributing

While these are personal notes, feedback is welcome!
- Found a typo? Open an issue
- Have a suggestion? Submit a pull request
- Want to discuss SQL? Reach out!

---

## 📧 Contact

**Author:** Edson Suárez  
**Focus:** Data Analytics & SQL Development  
**LinkedIn:** [Your LinkedIn]  
**GitHub:** [Your GitHub]

---

## 📜 Version History

### v2.0 (Current)
- Complete rewrite with improved formatting
- Enhanced tables and quick reference sections
- Expanded ADVANCED sections
- Added AI & SQL integration guide
- Removed incomplete notes and personal comments

### v1.0 (Original)
- Initial notes from YouTube course
- Raw format with some gaps
- Personal study comments included

---

> **"The only way to learn SQL is to write SQL."**  
> — Every SQL Expert Ever

---

*Last Updated: January 2025*  
*Course Source: [YouTube SQL Course]*  
*Documentation: Markdown + Git*
