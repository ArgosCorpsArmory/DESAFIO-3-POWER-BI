# DESAFIO-3-POWER-BI


Foi necessário upar o projeto em Azure SQL, para depois tratar os dados em tabela Power BI e responder 16 perguntas. Porém, achei mais válido, ao invés de dar upload em Azure, baixar o plugin POWER BI - MYSQL Command Line e integrar tudo através de comandos locais na minha máquina.
Também, upei os dados localmente, descobri q tinha uns erros de digitação como Department e Departament, alguns '' ao invés de ', entre outros detalhes de operação. Foi elaborado um método passo a passo para dar up nos dados no command line, detalhados a seguir:



ETAPA 1
<pre>
DROP SCHEMA IF EXISTS azure_company;
CREATE SCHEMA azure_company;
USE azure_company;
</pre>



ETAPA 2 CRIAR TABELA EMPLOYEE
<pre>
CREATE TABLE employee (
    Fname VARCHAR(15) NOT NULL,
    Minit CHAR(1),
    Lname VARCHAR(15) NOT NULL,
    Ssn CHAR(9) NOT NULL,
    Bdate DATE,
    Address VARCHAR(30),
    Sex CHAR(1),
    Salary DECIMAL(10,2),
    Super_ssn CHAR(9),
    Dno INT NOT NULL DEFAULT 1,
    CONSTRAINT pk_employee PRIMARY KEY (Ssn),
    CONSTRAINT chk_salary_employee CHECK (Salary > 2000.00)
);
</pre>


ETAPA 3 INSERIR FUNCIONARIOS
<pre>
INSERT INTO employee VALUES 
('John', 'B', 'Smith', '123456789', '1965-01-09', '731-Fondren-Houston-TX', 'M', 30000.00, '333445555', 5),
('Franklin', 'T', 'Wong', '333445555', '1955-12-08', '638-Voss-Houston-TX', 'M', 40000.00, '888665555', 5),
('Alicia', 'J', 'Zelaya', '999887777', '1968-01-19', '3321-Castle-Spring-TX', 'F', 25000.00, '987654321', 4),
('Jennifer', 'S', 'Wallace', '987654321', '1941-06-20', '291-Berry-Bellaire-TX', 'F', 43000.00, '888665555', 4),
('Ramesh', 'K', 'Narayan', '666884444', '1962-09-15', '975-Fire-Oak-Humble-TX', 'M', 38000.00, '333445555', 5),
('Joyce', 'A', 'English', '453453453', '1972-07-31', '5631-Rice-Houston-TX', 'F', 25000.00, '333445555', 5),
('Ahmad', 'V', 'Jabbar', '987987987', '1969-03-29', '980-Dallas-Houston-TX', 'M', 25000.00, '987654321', 4),
('James', 'E', 'Borg', '888665555', '1937-11-10', '450-Stone-Houston-TX', 'M', 55000.00, NULL, 1);
</pre>




ETAPA 4 CHAVE ESTRANGEIRA RECURSIVA SUPERVISOR
<pre>
ALTER TABLE employee
ADD CONSTRAINT fk_employee_supervisor FOREIGN KEY (Super_ssn) REFERENCES employee(Ssn)
ON DELETE SET NULL
ON UPDATE CASCADE;
</pre>


ETAPA 5 CRIAR E POPULAR DEPARTMENT
<pre>
CREATE TABLE department (
    Dname VARCHAR(15) NOT NULL,
    Dnumber INT NOT NULL,
    Mgr_ssn CHAR(9) NOT NULL,
    Mgr_start_date DATE,
    Dept_create_date DATE,
    CONSTRAINT pk_department PRIMARY KEY (Dnumber),
    CONSTRAINT unique_name_department UNIQUE (Dname),
    CONSTRAINT chk_date_department CHECK (Dept_create_date < Mgr_start_date),
    CONSTRAINT fk_department_mgr FOREIGN KEY (Mgr_ssn) REFERENCES employee(Ssn)
        ON UPDATE CASCADE
);

INSERT INTO department VALUES 
('Research', 5, '333445555', '1988-05-22', '1986-05-22'),
('Administration', 4, '987654321', '1995-01-01', '1994-01-01'),
('Headquarters', 1, '888665555', '1981-06-19', '1980-06-19');
</pre>




ETAPA 6 CRIAR E POPULAR DEPT_LOCATIONS
<pre>
CREATE TABLE dept_locations (
    Dnumber INT NOT NULL,
    Dlocation VARCHAR(15) NOT NULL,
    CONSTRAINT pk_dept_locations PRIMARY KEY (Dnumber, Dlocation),
    CONSTRAINT fk_dept_locations FOREIGN KEY (Dnumber) REFERENCES department(Dnumber)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

INSERT INTO dept_locations VALUES 
(1, 'Houston'),
(4, 'Stafford'),
(5, 'Bellaire'),
(5, 'Sugarland'),
(5, 'Houston');
</pre>




ETAPA 7 CRIAR E POPULAR PROJECT
<pre>
CREATE TABLE project (
    Pname VARCHAR(15) NOT NULL,
    Pnumber INT NOT NULL,
    Plocation VARCHAR(15),
    Dnum INT NOT NULL,
    CONSTRAINT pk_project PRIMARY KEY (Pnumber),
    CONSTRAINT unique_project_name UNIQUE (Pname),
    CONSTRAINT fk_project_department FOREIGN KEY (Dnum) REFERENCES department(Dnumber)
);

INSERT INTO project VALUES 
('ProductX', 1, 'Bellaire', 5),
('ProductY', 2, 'Sugarland', 5),
('ProductZ', 3, 'Houston', 5),
('Computerization', 10, 'Stafford', 4),
('Reorganization', 20, 'Houston', 1),
('Newbenefits', 30, 'Stafford', 4);
</pre>




ETAPA 8 CRIAR E POPULAR WORKS_ON
<pre>
CREATE TABLE works_on (
    Essn CHAR(9) NOT NULL,
    Pno INT NOT NULL,
    Hours DECIMAL(3,1) NOT NULL,
    CONSTRAINT pk_works_on PRIMARY KEY (Essn, Pno),
    CONSTRAINT fk_works_on_employee FOREIGN KEY (Essn) REFERENCES employee(Ssn),
    CONSTRAINT fk_works_on_project FOREIGN KEY (Pno) REFERENCES project(Pnumber)
);

INSERT INTO works_on VALUES 
('123456789', 1, 32.5),
('123456789', 2, 7.5),
('666884444', 3, 40.0),
('453453453', 1, 20.0),
('453453453', 2, 20.0),
('333445555', 2, 10.0),
('333445555', 3, 10.0),
('333445555', 10, 10.0),
('333445555', 20, 10.0),
('999887777', 30, 30.0),
('999887777', 10, 10.0),
('987987987', 10, 35.0),
('987987987', 30, 5.0),
('987654321', 30, 20.0),
('987654321', 20, 15.0),
('888665555', 20, 0.0);
</pre>

ETAPA 9 CRIAR E POPULAR DEPENDENT
<pre>
CREATE TABLE dependent (
    Essn CHAR(9) NOT NULL,
    Dependent_name VARCHAR(15) NOT NULL,
    Sex CHAR(1),
    Bdate DATE,
    Relationship VARCHAR(8),
    CONSTRAINT pk_dependent PRIMARY KEY (Essn, Dependent_name),
    CONSTRAINT fk_dependent_employee FOREIGN KEY (Essn) REFERENCES employee(Ssn)
);


INSERT INTO dependent VALUES 
('333445555', 'Alice', 'F', '1986-04-05', 'Daughter'),
('333445555', 'Theodore', 'M', '1983-10-25', 'Son'),
('333445555', 'Joy', 'F', '1958-05-03', 'Spouse'),
('987654321', 'Abner', 'M', '1942-02-28', 'Spouse'),
('123456789', 'Michael', 'M', '1988-01-04', 'Son'),
('123456789', 'Alice', 'F', '1988-12-30', 'Daughter'),
('123456789', 'Elizabeth', 'F', '1967-05-05', 'Spouse');
</pre>




Agora, houveram muitas perguntas, detalhando as respostas de cada uma com command line é mais divertido do que usar interface Power BI:

1. Verificar cabeçalhos e tipos de dados
<pre>
DESCRIBE employee;
DESCRIBE department;
DESCRIBE project;
DESCRIBE works_on;
DESCRIBE dependent;
DESCRIBE dept_locations;
</pre>
no employee, salary tá ('0,2) ok, os CHAR tipo SUPER SSN e tudo mais, tá CHAR(9), no power bi é possível q eu tenha que alterar para número int

2. Modificar valores monetários para DOUBLE
<pre>
ALTER TABLE employee MODIFY Salary DOUBLE;
</pre>
3. Verificar existência de nulos
<pre>
SELECT COUNT(*) FROM employee WHERE Super_ssn IS NULL;
SELECT COUNT(*) FROM department WHERE Mgr_ssn IS NULL;
</pre>
tem 1 Super_ssn nulo.... o que será que isso quer dizer? mais a frente vou descobrir (no próximo tópico número 4.)

4. Colaboradores sem gerente
<pre>
SELECT * FROM employee WHERE Super_ssn IS NULL;
</pre>
é o James? ele tem Ssn...... é o gerente? mas se tem colaborador sem gerente?

<pre>
SELECT e.Ssn
FROM employee e
LEFT JOIN employee s ON e.Super_ssn = s.Ssn
WHERE e.Super_ssn IS NOT NULL AND s.Ssn IS NULL;
</pre>
Não tem ninguém.... ou errei?


5. Departamentos sem gerente
<pre>
SELECT * FROM department d
LEFT JOIN employee e ON d.Mgr_ssn = e.Ssn
WHERE e.Ssn IS NULL;
</pre>
Não tem ninguém.... ou errei?


6. Preencher lacunas de gerente
UPDATE department SET Mgr_ssn = '888665555' WHERE Dnumber = 99;

7. Verificar número de horas por projeto
<pre>
SELECT Pno, SUM(Hours) AS Total_Hours
FROM works_on
GROUP BY Pno;
</pre>
<pre>
+-----+-------------+
| Pno | Total_Hours |
+-----+-------------+
|   1 |        52.5 |
|   2 |        37.5 |
|   3 |        50.0 |
|  10 |        55.0 |
|  20 |        25.0 |
|  30 |        55.0 |
+-----+-------------+
6 rows in set (0.056 sec)
</pre>

8. Separar colunas complexas
<pre>
SELECT 
  SUBSTRING_INDEX(Address, '-', 1) AS Street,
  SUBSTRING_INDEX(SUBSTRING_INDEX(Address, '-', 2), '-', -1) AS City,
  SUBSTRING_INDEX(Address, '-', -1) AS State
FROM employee;
</pre>
<pre>
+--------+---------+-------+
| Street | City    | State |
+--------+---------+-------+
| 731    | Fondren | TX    |
| 638    | Voss    | TX    |
| 5631   | Rice    | TX    |
| 975    | Fire    | TX    |
| 450    | Stone   | TX    |
| 291    | Berry   | TX    |
| 980    | Dallas  | TX    |
| 3321   | Castle  | TX    |
+--------+---------+-------+
8 rows in set (0.012 sec)
</pre>


9. Mesclar employee com department
<pre>
SELECT 
  e.Ssn,
  CONCAT(e.Fname, ' ', e.Lname) AS Employee_Name,
  d.Dname AS Department_Name
FROM employee e
LEFT JOIN department d ON e.Dno = d.Dnumber;
</pre>
<pre>
+-----------+------------------+-----------------+
| Ssn       | Employee_Name    | Department_Name |
+-----------+------------------+-----------------+
| 123456789 | John Smith       | Research        |
| 333445555 | Franklin Wong    | Research        |
| 453453453 | Joyce English    | Research        |
| 666884444 | Ramesh Narayan   | Research        |
| 888665555 | James Borg       | Headquarters    |
| 987654321 | Jennifer Wallace | Administration  |
| 987987987 | Ahmad Jabbar     | Administration  |
| 999887777 | Alicia Zelaya    | Administration  |
+-----------+------------------+-----------------+
8 rows in set (0.008 sec)
</pre>


10. Eliminar colunas desnecessárias
<pre>
SELECT 
  e.Ssn,
  CONCAT(e.Fname, ' ', e.Lname) AS Employee_Name,
  d.Dname AS Department_Name
FROM employee e
LEFT JOIN department d ON e.Dno = d.Dnumber;
</pre>
<pre>
+-----------+------------------+-----------------+
| Ssn       | Employee_Name    | Department_Name |
+-----------+------------------+-----------------+
| 123456789 | John Smith       | Research        |
| 333445555 | Franklin Wong    | Research        |
| 453453453 | Joyce English    | Research        |
| 666884444 | Ramesh Narayan   | Research        |
| 888665555 | James Borg       | Headquarters    |
| 987654321 | Jennifer Wallace | Administration  |
| 987987987 | Ahmad Jabbar     | Administration  |
| 999887777 | Alicia Zelaya    | Administration  |
+-----------+------------------+-----------------+
8 rows in set (0.011 sec)
</pre>


11. Junção com nome do gerente
<pre>
SELECT 
  e.Ssn,
  CONCAT(e.Fname, ' ', e.Lname) AS Employee_Name,
  CONCAT(m.Fname, ' ', m.Lname) AS Manager_Name
FROM employee e
LEFT JOIN employee m ON e.Super_ssn = m.Ssn;
</pre>
<pre>
+-----------+------------------+------------------+
| Ssn       | Employee_Name    | Manager_Name     |
+-----------+------------------+------------------+
| 123456789 | John Smith       | Franklin Wong    |
| 333445555 | Franklin Wong    | James Borg       |
| 453453453 | Joyce English    | Franklin Wong    |
| 666884444 | Ramesh Narayan   | Franklin Wong    |
| 888665555 | James Borg       | NULL             |
| 987654321 | Jennifer Wallace | James Borg       |
| 987987987 | Ahmad Jabbar     | Jennifer Wallace |
| 999887777 | Alicia Zelaya    | Jennifer Wallace |
+-----------+------------------+------------------+
8 rows in set (0.009 sec)
</pre>

12. Mesclar nome e sobrenome
<pre>
SELECT CONCAT(Fname, ' ', Lname) AS Full_Name FROM employee;
</pre>
<pre>
+------------------+
| Full_Name        |
+------------------+
| John Smith       |
| Franklin Wong    |
| Joyce English    |
| Ramesh Narayan   |
| James Borg       |
| Jennifer Wallace |
| Ahmad Jabbar     |
| Alicia Zelaya    |
+------------------+
8 rows in set (0.011 sec)
</pre>

13. Mesclar nome de departamento e localização
<pre>
SELECT 
  d.Dname,
  l.Dlocation,
  CONCAT(d.Dname, ' - ', l.Dlocation) AS Dept_Location
FROM department d
JOIN dept_locations l ON d.Dnumber = l.Dnumber;
</pre>
<pre>
+----------------+-----------+---------------------------+
| Dname          | Dlocation | Dept_Location             |
+----------------+-----------+---------------------------+
| Administration | Stafford  | Administration - Stafford |
| Headquarters   | Houston   | Headquarters - Houston    |
| Research       | Bellaire  | Research - Bellaire       |
| Research       | Houston   | Research - Houston        |
| Research       | Sugarland | Research - Sugarland      |
+----------------+-----------+---------------------------+
5 rows in set (0.008 sec)
</pre>

14. Por que usar mesclar e não atribuir
👉 Mesclar une dados de duas tabelas com base em uma chave comum (join).
👉 Atribuir substitui ou atualiza valores em uma coluna existente.

pra preserver os dados originais = Mesclar


15. Agrupar colaboradores por gerente
<pre>
SELECT 
  Super_ssn AS Manager_Ssn,
  COUNT(*) AS Num_Employees
FROM employee
WHERE Super_ssn IS NOT NULL
GROUP BY Super_ssn;
</pre>
<pre>
+-------------+---------------+
| Manager_Ssn | Num_Employees |
+-------------+---------------+
| 333445555   |             3 |
| 888665555   |             2 |
| 987654321   |             2 |
+-------------+---------------+
3 rows in set (0.010 sec)
</pre>

16. Eliminar colunas desnecessárias
<pre>
SELECT 
  Ssn,
  CONCAT(Fname, ' ', Lname) AS Employee_Name,
  Dno
FROM employee;
</pre>
<pre>
+-----------+------------------+-----+
| Ssn       | Employee_Name    | Dno |
+-----------+------------------+-----+
| 123456789 | John Smith       |   5 |
| 333445555 | Franklin Wong    |   5 |
| 453453453 | Joyce English    |   5 |
| 666884444 | Ramesh Narayan   |   5 |
| 888665555 | James Borg       |   1 |
| 987654321 | Jennifer Wallace |   4 |
| 987987987 | Ahmad Jabbar     |   4 |
| 999887777 | Alicia Zelaya    |   4 |
+-----------+------------------+-----+
8 rows in set (0.008 sec)
</pre>






