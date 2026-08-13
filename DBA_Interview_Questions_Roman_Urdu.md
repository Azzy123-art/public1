# SQL Server DBA Interview Questions & Answers (Roman Urdu)

Yeh file mein DBA interview ke sab important topics cover kiye gaye hain: High Availability, Failover Clustering, Indexes, Execution Plans, Slow Running Queries, Backups, aur baqi core concepts. Answers Roman Urdu mein hain taake asani se samajh aa sake.

---

## SECTION 1: HIGH AVAILABILITY (HA) & DISASTER RECOVERY (DR)

### Q1. High Availability (HA) kya hoti hai?
**Answer:** High Availability ka matlab hai ke aapka database system maximum time available rahe, chahe koi hardware failure ho, server crash ho ya maintenance chal rahi ho. HA ka goal downtime ko kam se kam karna hai. Isay usually "nines" mein measure karte hain — jaise 99.9% (three nines) matlab saal mein sirf ~8.7 ghantay downtime allowed hai, aur 99.99% matlab sirf ~52 minutes.

### Q2. SQL Server mein HA/DR ke kaun kaun se options hote hain?
**Answer:** SQL Server mein yeh main options hain:
1. **Always On Availability Groups (AG)** – Modern aur sab se popular solution. Database level protection deta hai.
2. **Failover Cluster Instance (FCI)** – Instance level protection, shared storage use karta hai.
3. **Log Shipping** – Transaction log backups doosre server pe restore hote rehte hain. Simple aur cheap DR solution.
4. **Database Mirroring** – Purana feature (deprecated), ek principal aur ek mirror server hota tha.
5. **Replication** – Data distribute karne ke liye (Snapshot, Transactional, Merge). Yeh technically HA nahi hai, data distribution solution hai.

### Q3. RTO aur RPO kya hote hain?
**Answer:**
- **RTO (Recovery Time Objective):** Failure ke baad system ko wapas online lane mein kitna time lag sakta hai. Matlab kitni der ka downtime bardasht kar sakte ho.
- **RPO (Recovery Point Objective):** Kitna data loss bardasht kar sakte ho. Agar RPO 15 minutes hai, to aakhri 15 minutes ka data loss acceptable hai.
Yeh dono business requirements se decide hote hain aur inhi ki base par HA/DR solution choose kiya jata hai.

### Q4. Always On Availability Group (AG) kya hai aur kaise kaam karta hai?
**Answer:** Always On AG SQL Server ka enterprise-level HA solution hai. Ismein:
- Ek **Primary Replica** hoti hai jahan read/write hota hai.
- Ek ya zyada **Secondary Replicas** hoti hain jahan data continuously sync hota rehta hai.
- Databases ka ek group (Availability Group) ek sath failover hota hai.
- Client applications **Listener** ke through connect karti hain, jo automatically primary replica pe redirect kar deta hai.
- Yeh Windows Server Failover Cluster (WSFC) ke upar chalta hai lekin shared storage ki zaroorat nahi hoti — har replica ki apni copy hoti hai.

### Q5. Synchronous aur Asynchronous commit mode mein kya farq hai?
**Answer:**
- **Synchronous Commit:** Primary tab tak transaction commit nahi karta jab tak secondary confirm na kare ke log receive ho gaya. Zero data loss milta hai lekin thori performance cost hoti hai. Automatic failover isi mode mein possible hai. Usually same datacenter ke servers ke liye use hota hai.
- **Asynchronous Commit:** Primary commit kar deta hai bina secondary ka wait kiye. Performance behtar hoti hai lekin failover pe kuch data loss ho sakta hai. Usually DR site (door wale datacenter) ke liye use hota hai.

### Q6. Log Shipping kya hai aur kab use karte hain?
**Answer:** Log Shipping mein primary server pe transaction log backups lete hain, unhe secondary server pe copy karte hain, aur wahan restore karte rehte hain. Teen jobs hoti hain: **Backup job** (primary pe), **Copy job** aur **Restore job** (secondary pe). Yeh cheap aur simple DR solution hai, lekin failover **manual** hota hai aur data loss ho sakta hai (jitna log backup interval hai). Secondary ko read-only reporting ke liye bhi use kar sakte hain (STANDBY mode mein).

### Q7. Database Mirroring aur AG mein kya farq hai?
**Answer:** Mirroring purana feature hai jo ab deprecated hai. Ismein sirf ek database ek waqt mein mirror hota hai aur sirf ek secondary ho sakti hai jo readable nahi hoti. AG mein multiple databases ek group mein failover ho sakte hain, multiple secondaries ho sakti hain (SQL 2019 mein 8 tak), aur secondaries **readable** hoti hain jahan reporting aur backups offload kar sakte hain.

---

## SECTION 2: FAILOVER CLUSTERING (FCI / WSFC)

### Q8. Failover Cluster Instance (FCI) kya hai?
**Answer:** FCI mein SQL Server ka poora instance Windows Server Failover Cluster (WSFC) pe install hota hai. Do ya zyada nodes hote hain lekin **storage shared** hota hai (SAN). Ek waqt mein sirf ek node active hota hai. Agar active node fail ho jaye, to SQL Server services automatically doosre node pe start ho jati hain aur wahi shared storage attach ho jata hai. Client ek **Virtual Network Name (VNN)** se connect karta hai, is liye application ko pata bhi nahi chalta ke failover hua.

### Q9. FCI aur Always On AG mein kya farq hai?
**Answer:**
| Point | FCI | AG |
|---|---|---|
| Protection level | Poora instance | Database group |
| Storage | Shared (SAN) — single point of failure | Har replica ki apni copy |
| Data copies | Ek hi copy | Multiple copies |
| Readable secondary | Nahi | Haan |
| Licensing/System DBs | System databases, logins, jobs sab automatically protected | Logins/jobs manually sync karne parte hain |
FCI local HA ke liye acha hai, AG local HA + DR + read scaling deta hai. Bohat companies dono combine karti hain (FCI + AG async DR replica).

### Q10. Quorum kya hota hai clustering mein?
**Answer:** Quorum ek voting mechanism hai jo decide karta hai ke cluster chalta rahe ya band ho jaye. Har node ka ek vote hota hai, aur majority (aadha se zyada) votes chahiye hote hain cluster ko online rehne ke liye. Iska maqsad **split-brain** situation rokna hai — matlab yeh na ho ke network partition ki wajah se dono nodes khud ko primary samajh lein aur data corrupt ho jaye. Quorum ke types: Node Majority, Node + Disk Witness, Node + File Share Witness, aur **Cloud Witness** (Azure).

### Q11. Split-brain scenario kya hota hai?
**Answer:** Jab cluster nodes ke darmiyan network communication toot jaye aur dono sides khud ko active samajh kar independently kaam karna shuru kar dein — is se same database pe do jagah writes hone lagti hain aur data corrupt/inconsistent ho jata hai. Quorum mechanism isi ko rokta hai: jis side ke paas majority votes nahi hote, wo side khud ko shut down kar leti hai.

### Q12. Failover ke types kya hain?
**Answer:**
1. **Automatic Failover:** Synchronous mode + automatic failover setting pe hota hai. Health issue detect hote hi cluster khud failover kar deta hai.
2. **Manual (Planned) Failover:** DBA khud initiate karta hai, jaise patching se pehle. Zero data loss.
3. **Forced Failover (with possible data loss):** Jab primary bilkul down ho aur async secondary ko force se primary banana pare. Data loss ka risk hota hai.

### Q13. AG Listener kya hota hai?
**Answer:** Listener ek virtual network name (VNN) aur IP hota hai jis se applications connect karti hain. Failover ke baad listener automatically naye primary ki taraf point kar deta hai, is liye application ki connection string change nahi karni parti. Read-only routing ke sath listener read requests ko readable secondary pe bhi bhej sakta hai (`ApplicationIntent=ReadOnly`).

---

## SECTION 3: INDEXES

### Q14. Index kya hota hai aur kyun use karte hain?
**Answer:** Index kitab ke index ki tarah hota hai — data jaldi dhoondhne ke liye. Bina index ke SQL Server poori table scan karta hai (Table Scan) jo bari tables pe bohat slow hota hai. Index B-Tree structure mein data organize karta hai jis se lookups fast ho jate hain. Lekin har index ki cost hoti hai: INSERT/UPDATE/DELETE slow hote hain kyunke index bhi maintain karna parta hai, aur storage bhi extra lagta hai.

### Q15. Clustered aur Non-Clustered index mein kya farq hai?
**Answer:**
- **Clustered Index:** Table ka actual data isi order mein physically store hota hai. Ek table pe sirf **ek** clustered index ho sakta hai. Usually Primary Key pe hota hai. Leaf level pe actual data pages hoti hain.
- **Non-Clustered Index:** Alag structure hota hai jo data ki taraf pointer rakhta hai (clustered key ya RID). Ek table pe multiple (999 tak) non-clustered indexes ho sakte hain. Leaf level pe sirf key columns + pointer hota hai.
Misal: Clustered index phone book ki tarah hai (data khud sorted hai), non-clustered kitab ke pichay wala index hai (topic ka page number batata hai).

### Q16. Covering Index kya hota hai?
**Answer:** Jab index mein query ke sare required columns maujood hon — chahe key columns mein hon ya **INCLUDE** columns mein — to SQL Server ko table pe wapas jane (Key Lookup) ki zaroorat nahi parti. Isay covering index kehte hain. Example:
```sql
CREATE INDEX IX_Orders_CustomerID
ON Orders (CustomerID)
INCLUDE (OrderDate, TotalAmount);
```
Ab `SELECT OrderDate, TotalAmount FROM Orders WHERE CustomerID = 5` poori tarah index se serve ho jayegi.

### Q17. Key Lookup kya hota hai aur isay kaise fix karte hain?
**Answer:** Jab non-clustered index se rows mil jati hain lekin query ko kuch aur columns bhi chahiye hote hain jo index mein nahi hain, to SQL Server har row ke liye clustered index/heap pe wapas jata hai — isay **Key Lookup** (ya RID Lookup heap pe) kehte hain. Agar rows zyada hon to yeh bohat expensive hota hai. Fix: zaroori columns ko index mein **INCLUDE** kar do (covering index bana do).

### Q18. Index Fragmentation kya hai aur kaise theek karte hain?
**Answer:** Jab data insert/update/delete hota hai to index pages disorder ho jati hain — logical order aur physical order match nahi karta. Isay fragmentation kehte hain, jo scans slow kar deti hai.
- **Check:** `sys.dm_db_index_physical_stats` DMV se.
- **Fix:**
  - Fragmentation **5–30%** ho to `ALTER INDEX ... REORGANIZE` (online, halka operation).
  - **30% se zyada** ho to `ALTER INDEX ... REBUILD` (index dobara banta hai; Enterprise edition mein ONLINE = ON kar sakte hain).

### Q19. Fill Factor kya hota hai?
**Answer:** Fill factor batata hai ke index page kitni percent bharni hai jab index build/rebuild ho. Agar fill factor 90 hai to har page mein 10% jagah khali chorhi jayegi taake future inserts ke liye room ho aur **page splits** kam hon. Page split expensive operation hai jo fragmentation barhata hai. Heavy insert wali tables pe 80–90 fill factor rakhte hain; read-only tables pe 100.

### Q20. Filtered Index kya hai?
**Answer:** Non-clustered index jo sirf specific rows pe banta hai, WHERE clause ke sath. Example:
```sql
CREATE INDEX IX_Orders_Active ON Orders (OrderDate)
WHERE Status = 'Active';
```
Chhota index, kam maintenance, aur specific queries ke liye bohat fast. Sparse data (jaise NULL ya ek particular status) ke liye best hai.

### Q21. Missing indexes kaise dhoondte hain?
**Answer:** SQL Server missing index recommendations record karta hai in DMVs mein: `sys.dm_db_missing_index_details`, `sys.dm_db_missing_index_group_stats`. Execution plan mein bhi green text mein "Missing Index" suggestion aata hai. Lekin in suggestions ko blindly apply nahi karna chahiye — pehle check karo ke existing index modify ho sakta hai, duplicate na banao, aur write workload ka impact socho.

### Q22. Unused indexes ka kya karna chahiye?
**Answer:** `sys.dm_db_index_usage_stats` se dekho ke index pe seeks/scans kitne hain vs updates kitne hain. Agar index kabhi read nahi hota lekin har write pe update hota hai, to wo sirf overhead hai — usay drop kar dena chahiye (pehle script save kar lo). Note: yeh stats SQL restart pe reset ho jate hain, is liye kaafi time ka data dekh kar faisla karo.

---

## SECTION 4: EXECUTION PLANS

### Q23. Execution Plan kya hota hai?
**Answer:** Execution plan wo roadmap hai jo SQL Server ka Query Optimizer banata hai — yeh batata hai ke query kaise execute hogi: kaunse indexes use honge, joins kaise honge, kitni rows expect hain, waghera. Do types:
- **Estimated Plan:** Query run kiye baghair (Ctrl+L) — optimizer ka andaza.
- **Actual Plan:** Query run karke (Ctrl+M enable karke) — asal numbers ke sath (actual rows, actual executions).

### Q24. Table Scan, Index Scan aur Index Seek mein kya farq hai?
**Answer:**
- **Table Scan:** Poori heap table parh li jati hai. Bari table pe sab se slow.
- **Index Scan (Clustered Index Scan):** Poora index start se end tak parha jata hai. Yeh bhi mehnga hai agar sirf chand rows chahiye thin.
- **Index Seek:** B-Tree ke through directly required rows tak pohanchna. Sab se efficient. Goal usually yeh hota hai ke selective queries seek use karein.
Note: chhoti tables pe scan bhi theek hai; seek hamesha behtar nahi hota agar zyada rows chahiye hon.

### Q25. Nested Loops, Merge Join aur Hash Join kab use hote hain?
**Answer:**
- **Nested Loop Join:** Chhota outer input aur indexed inner input ho — har outer row ke liye inner mein seek. Small result sets pe best.
- **Merge Join:** Dono inputs join key pe **sorted** hon. Bari sorted inputs ke liye efficient.
- **Hash Join:** Bari, unsorted inputs. Ek input se hash table banti hai, doosra probe hota hai. Memory-heavy hai; large data warehouse queries mein common.
Agar chhote data pe hash join dikhe ya bari table pe nested loop laakhon baar chale, to statistics ya indexing ka masla ho sakta hai.

### Q26. Estimated vs Actual rows ka farq kyun important hai?
**Answer:** Agar optimizer ne estimate kiya 100 rows lekin actually 1 million aayin, to iska matlab **statistics outdated hain** ya **parameter sniffing** ho rahi hai ya cardinality estimation ka issue hai. Ghalat estimates ki wajah se ghalat join type, ghalat memory grant, aur spills to tempdb hote hain. Fix: statistics update karo (`UPDATE STATISTICS` ya `sp_updatestats`), query rewrite karo, ya hints use karo.

### Q27. Parameter Sniffing kya hai?
**Answer:** Jab stored procedure pehli baar chalta hai, SQL Server us waqt ke parameter values ke hisaab se plan banata hai aur cache kar leta hai. Agar wo parameter atypical tha (jaise aisi value jis ki sirf 10 rows hain), to wahi plan baad mein aisi values ke liye bhi use hota hai jin ki 10 million rows hain — aur query bohat slow ho jati hai. **Fixes:** `OPTION (RECOMPILE)`, `OPTIMIZE FOR` hint, local variables, ya SQL 2022 ka Parameter Sensitive Plan (PSP) optimization.

### Q28. Execution plan mein kin cheezon pe focus karna chahiye?
**Answer:**
1. Sab se **expensive operators** (highest cost %).
2. **Thick arrows** — zyada rows flow ho rahi hain.
3. **Scans** wahan jahan seek hona chahiye tha.
4. **Key Lookups** with high executions.
5. **Warnings** (yellow triangle) — implicit conversions, spills to tempdb, missing statistics.
6. **Estimated vs Actual rows** ka bara farq.
7. **Sort** aur **Hash** operators jo memory spill kar rahe hon.

### Q29. Implicit Conversion kya hai aur kyun buri hai?
**Answer:** Jab column ka data type aur compare ki jane wali value ka type match nahi karta (jaise `VARCHAR` column ko `NVARCHAR` parameter se compare karna), to SQL Server column pe conversion apply karta hai. Is se index seek nahi ho sakta — scan hota hai. Execution plan mein warning dikhti hai (`CONVERT_IMPLICIT`). Fix: parameter/variable ka data type column ke type se match karo.

---

## SECTION 5: SLOW RUNNING QUERIES (PERFORMANCE TROUBLESHOOTING)

### Q30. Koi query slow hai — troubleshooting kaise start karoge?
**Answer:** Step by step approach:
1. **Confirm karo** query khud slow hai ya blocking/waiting mein hai — `sp_WhoIsActive` ya `sys.dm_exec_requests` se dekho.
2. **Wait type check karo** — kya wait hai? CXPACKET, PAGEIOLATCH, LCK_M_X, etc.
3. **Actual execution plan** lo aur scans, key lookups, warnings, estimate vs actual dekho.
4. **Statistics** updated hain ya nahi check karo.
5. **Indexes** — missing ya fragmented?
6. **Query rewrite** — SARGable predicates, unnecessary SELECT *, cursors ko set-based mein convert karna.
7. **Server level** — CPU, memory pressure, tempdb contention, IO latency.

### Q31. Blocking aur Deadlock mein kya farq hai?
**Answer:**
- **Blocking:** Ek session ne lock le rakha hai aur doosra wait kar raha hai. Yeh normal hai; jab pehla commit karega to doosra chal parega. Lamba blocking problem hai.
- **Deadlock:** Do sessions ek doosre ke locked resources ka intezar kar rahe hain — circular wait. Koi bhi aage nahi barh sakta. SQL Server khud detect karke ek session ko **victim** bana kar kill kar deta hai (error 1205).
**Deadlock fix:** Transactions chhoti rakho, objects ko same order mein access karo, proper indexes lagao taake locks kam hon, aur zaroorat ho to RCSI (Read Committed Snapshot Isolation) consider karo.

### Q32. Wait Statistics kya hain aur common wait types kya batate hain?
**Answer:** Wait stats (`sys.dm_os_wait_stats`) batate hain ke SQL Server kis cheez ka intezar kar raha hai:
- **PAGEIOLATCH_SH/EX:** Disk IO slow hai ya memory kam hai (data disk se parhna par raha hai).
- **LCK_M_*:** Locking/blocking issues.
- **CXPACKET / CXCONSUMER:** Parallelism waits — MAXDOP aur cost threshold check karo.
- **WRITELOG:** Transaction log disk slow hai.
- **RESOURCE_SEMAPHORE:** Memory grants ka intezar — bari queries memory ke liye queue mein hain.
- **PAGELATCH_* (tempdb pe):** Tempdb contention — multiple tempdb data files banao.

### Q33. SARGable query kya hoti hai?
**Answer:** SARGable = Search ARGument-able, matlab aisi query jis pe index seek ho sakta hai. Agar WHERE clause mein column pe function laga do to index use nahi hota:
- **Bad:** `WHERE YEAR(OrderDate) = 2025` (non-SARGable)
- **Good:** `WHERE OrderDate >= '2025-01-01' AND OrderDate < '2026-01-01'`
- **Bad:** `WHERE LEFT(Name, 3) = 'ABC'`
- **Good:** `WHERE Name LIKE 'ABC%'`
Column pe function, calculation, ya leading wildcard (`LIKE '%abc'`) — yeh sab index seek rok dete hain.

### Q34. Statistics kya hain aur kyun important hain?
**Answer:** Statistics data distribution ki information hoti hai (histogram) jo optimizer estimates ke liye use karta hai. Agar stats purani hon to optimizer ghalat row estimates karega aur ghalat plan banega. Auto update statistics usually ON hota hai, lekin bari tables pe threshold late hit hota hai, is liye regular maintenance job mein `sp_updatestats` ya Ola Hallengren ke scripts se stats update karte hain.

### Q35. Query Store kya hai?
**Answer:** Query Store (SQL 2016+) ek flight recorder hai jo queries, unke plans aur runtime stats history save karta hai. Is se:
- Plan regression pakar sakte ho (query pehle fast thi, ab slow kyun?).
- Purana acha plan **force** kar sakte ho.
- Top resource consuming queries dekh sakte ho.
Enable: `ALTER DATABASE [db] SET QUERY_STORE = ON;`

### Q36. sp_WhoIsActive kya hai?
**Answer:** Adam Machanic ka free community stored procedure jo real-time mein batata hai ke server pe abhi kya chal raha hai: active queries, wait types, blocking chains, tempdb usage, CPU, reads. DBA troubleshooting ka pehla tool yehi hota hai. Built-in alternative: `sp_who2` (basic) ya `sys.dm_exec_requests` + `sys.dm_exec_sessions` DMVs.

### Q37. MAXDOP aur Cost Threshold for Parallelism kya hain?
**Answer:**
- **MAXDOP (Max Degree of Parallelism):** Ek query maximum kitne CPU cores use kar sakti hai. Default 0 (sare cores) — OLTP pe usually 8 ya NUMA node ke hisaab se set karte hain.
- **Cost Threshold for Parallelism:** Query ka estimated cost is number se zyada ho tab hi parallel plan banega. Default 5 bohat purana/kam hai — usually 25–50 set karte hain taake chhoti queries faltu parallel na hon.

### Q38. TempDB contention kya hai aur fix kya hai?
**Answer:** Jab bohat si sessions ek sath temp tables banati hain to tempdb ke allocation pages (PFS, GAM, SGAM) pe latch contention hoti hai (PAGELATCH waits). **Fix:** Multiple tempdb data files banao (CPU cores ke hisaab se, usually 4–8, sab equal size), sab files same size aur same autogrowth rakho. SQL 2016+ setup khud multiple files suggest karta hai.

---

## SECTION 6: BACKUP & RECOVERY

### Q39. Backup ke types kya hain?
**Answer:**
1. **Full Backup:** Poora database. Baseline hoti hai.
2. **Differential Backup:** Aakhri full backup ke baad jo change hua sirf wo. Restore ke liye: last full + last differential.
3. **Transaction Log Backup:** Log records backup hote hain. Sirf FULL/BULK_LOGGED recovery model mein possible. Point-in-time recovery isi se hoti hai.
4. **Copy-Only Backup:** Backup chain ko disturb kiye baghair ad-hoc backup.

### Q40. Recovery Models kya hain?
**Answer:**
- **SIMPLE:** Log automatically truncate hota hai. Log backup nahi ho sakta, point-in-time recovery nahi. Dev/test ke liye.
- **FULL:** Sab kuch log hota hai, log backups zaroori hain warna log file barhti jayegi. Point-in-time recovery possible. Production standard.
- **BULK_LOGGED:** Bulk operations minimally log hote hain. Bulk load ke waqt temporarily use karte hain.

### Q41. Point-in-time recovery kaise karte hain?
**Answer:** Last full backup restore karo `WITH NORECOVERY`, phir last differential `WITH NORECOVERY`, phir sare log backups sequence mein `WITH NORECOVERY`, aur aakhri log backup pe `WITH STOPAT = 'exact time'` aur `RECOVERY` use karo. Example scenario: kisi ne ghalti se 2:30 PM pe table delete kar di — aap 2:29 PM tak restore kar sakte ho.

### Q42. Backup verify kaise karte ho?
**Answer:** Sirf backup lena kaafi nahi — restore test karna asal verification hai. Options: `RESTORE VERIFYONLY` (basic check), backup mein `WITH CHECKSUM` use karna, aur best practice: regularly kisi test server pe restore karke `DBCC CHECKDB` chalana. "Backup utna hi acha hai jitna aakhri successful restore."

---

## SECTION 7: GENERAL / MISCELLANEOUS DBA QUESTIONS

### Q43. DBCC CHECKDB kya karta hai?
**Answer:** Database ki physical aur logical integrity check karta hai — corruption dhoondta hai. Regularly (weekly at least) chalana chahiye. Agar corruption mile to pehli choice hamesha **backup se restore** hai; `REPAIR_ALLOW_DATA_LOSS` aakhri option hai kyunke ismein data loss hota hai.

### Q44. Isolation Levels kya hain?
**Answer:**
1. **READ UNCOMMITTED:** Dirty reads allowed (NOLOCK). Fastest lekin inconsistent data mil sakta hai.
2. **READ COMMITTED:** Default. Sirf committed data parhta hai.
3. **REPEATABLE READ:** Transaction ke dauran parhi hui rows change nahi ho saktin.
4. **SERIALIZABLE:** Sab se strict — phantom rows bhi nahi aa saktin. Sab se zyada blocking.
5. **SNAPSHOT / RCSI:** Row versioning (tempdb) use karke readers writers ko block nahi karte. Blocking-heavy systems mein RCSI bohat useful hai.

### Q45. NOLOCK hint use karna theek hai?
**Answer:** NOLOCK (READ UNCOMMITTED) dirty reads deta hai — aap uncommitted, duplicate ya missing rows parh sakte ho. Financial ya accurate reports ke liye bilkul nahi. Agar blocking ka masla hai to behtar solution **RCSI** enable karna hai jo consistent data deta hai baghair readers ko block kiye.

### Q46. SQL Server memory kaise manage karta hai? Max Server Memory kya set karni chahiye?
**Answer:** SQL Server buffer pool mein data pages cache karta hai aur jitni memory mile le leta hai. **Max Server Memory** set karna zaroori hai taake OS ke liye memory bache. Rule of thumb: OS ke liye 4 GB + har additional 8–16 GB pe 1 GB chorho (ya total ka ~10–20%). Example: 64 GB server pe max memory ~56–58 GB.

### Q47. Page Life Expectancy (PLE) kya hai?
**Answer:** PLE batata hai ke data page buffer pool mein average kitne seconds rehta hai. Kam PLE ka matlab memory pressure — pages jaldi flush ho rahe hain aur disk se dobara parhne par rahe hain. Purana benchmark 300 seconds tha, lekin modern servers pe per 4GB memory ~300 seconds ka formula behtar hai. PLE ko trend ke tor pe dekho, single number ke tor pe nahi.

### Q48. Maintenance plan mein kya kya hona chahiye?
**Answer:**
1. **Backups:** Full (daily/weekly), Differential, Log backups (har 15–30 min).
2. **Index maintenance:** Reorganize/Rebuild fragmentation ke hisaab se.
3. **Statistics update.**
4. **DBCC CHECKDB** (integrity check).
5. **Cleanup:** Purani backup files aur history.
Industry standard: **Ola Hallengren ke maintenance scripts** — interview mein yeh naam lena plus point hai.

### Q49. SQL Server Agent job fail ho jaye to kaise troubleshoot karoge?
**Answer:** Job history dekho (job pe right-click > View History), error message parho, job ke step ki output file check karo, SQL Server Agent error log aur SQL Server error log dekho. Common issues: permissions (job owner/proxy account), network/path issues, blocking ya timeout. Alerts aur notifications (operator + email) configure hone chahiye taake failure ka foran pata chale.

### Q50. Ek production server achanak slow ho gaya — pehle 5 minute mein kya karoge?
**Answer:**
1. `sp_WhoIsActive` chalao — kya running hai, koi blocking chain to nahi?
2. Wait stats dekho — CPU, IO, memory, ya locking ka issue?
3. CPU/Memory usage check karo (Task Manager / PerfMon / DMVs).
4. Koi recent change? Deployment, index rebuild, stats change, ya koi ad-hoc bari query?
5. Error log dekho — memory dumps, IO warnings ("IO taking longer than 15 seconds"), failovers.
6. Agar ek hi query culprit hai to uska plan dekho; agar blocking hai to head blocker identify karo aur zaroorat pe (approval ke sath) kill karo.

---

## BONUS: Quick One-Liners (Rapid Fire)

- **Heap kya hai?** Aisi table jis pe clustered index nahi hota.
- **Primary Key vs Unique Key?** PK ek hi ho sakti hai aur NULL allow nahi; Unique key multiple ho sakti hain aur ek NULL allow karti hai.
- **DELETE vs TRUNCATE?** DELETE row-by-row log hota hai, WHERE laga sakte ho; TRUNCATE page deallocation hai, fast hai, identity reset karta hai.
- **VLF kya hain?** Virtual Log Files — transaction log ke andar ke sections. Bohat zyada VLFs recovery slow karte hain.
- **Orphaned users?** Jab login aur database user ka SID match na kare (usually restore ke baad). Fix: `ALTER USER ... WITH LOGIN = ...`
- **Checkpoint kya karta hai?** Dirty pages memory se disk pe likhta hai.
- **Lazy Writer?** Memory pressure pe buffer pool se pages free karta hai.
- **sp_recompile kab?** Jab kisi proc ka cached plan force se dobara banwana ho.
- **AlwaysOn readable secondary ka faida?** Reporting aur backups offload kar sakte ho, primary pe load kam.
- **In-place upgrade vs side-by-side?** Side-by-side (naya server, migrate) safer aur preferred hai.

---

*Best of luck for your interview! In topics ko sirf ratta na lagao — har concept ko apne lafzon mein explain karne ki practice karo, kyunke interviewer follow-up questions zaroor poochta hai.*
