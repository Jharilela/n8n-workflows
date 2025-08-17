This workflow automatically backs up all public Postgres tables into a GitHub repository as CSV files every 24 hours.  
It ensures your database snapshots are always up to date updating existing files if data changes, or creating new backups for new tables.  

This workflow is also published on the [n8n official site](https://n8n.io/workflows/7419-daily-postgres-table-backup-to-github-in-csv-format/)

## 📺 Video Tutorial

[![n8n Discord Bot Tutorial](https://img.youtube.com/vi/ZECpVg8n7-Y/maxresdefault.jpg)](https://youtu.be/ZECpVg8n7-Y)

*Watch our complete tutorial on how to set up and use the n8n Discord trigger bot*

---


**How it works:**  
1. **Schedule Trigger** – Runs daily to start the backup process.  
2. **GitHub Integration** – Lists existing files in the target repo to avoid duplicates.  
3. **Postgres Query** – Fetches all table names from the `public` schema.  
4. **Data Extraction** – Selects all rows from each table.  
5. **Convert to CSV** – Saves table data as CSV files.  
6. **Conditional Upload** –  
   - If the table already exists in GitHub → Update the file.  
   - If new → Upload a new file.  

---

### **Postgres Tables Preview**
![postgres table](https://articles.emp0.com/wp-content/uploads/2025/08/backup-postgres-to-github-tables.png)

---

### **GitHub Backup Preview**
![github backup](https://articles.emp0.com/wp-content/uploads/2025/08/backup-postgres-to-github-repo.png)

---

**Use case:**  
Perfect for developers, analysts, or data engineers who want **daily automated backups** of Postgres data without manual exports keeping both history and version control in GitHub.  

**Requirements:**  
- Postgres credentials with read access.  
- GitHub repository (OAuth2 connected in n8n).  
