# sqlserver-db

**Name**
```plain text
sqlserver-db
```
**YAML**
```yaml
version: '3.4' 
 
services: 
  sqlserverdb: 
    image: mcr.microsoft.com/mssql/server:2019-latest 
    container_name : sqlserverdb 
    restart: always 
    environment: 
      - SA_PASSWORD=7BevVwyfj9rrhAuU7emC
      - ACCEPT_EULA=Y 
    ports: 
      - "1433:1433"
    volumes:
      - hikvision_data:/var/opt/mssql  
volumes:
  hikvision_data:   

```
**Create Table**
```sql
USE [hikvisiondb]
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[TabEmployeeAttendance](
  [EmployeeID] [varchar](50) NULL,
  [AuthenticationDateAndTime] [datetime] NULL,
  [AuthenticationDate] [date] NULL,
  [AuthenticationTime] [time](7) NULL,
  [Direction] [varchar](50) NULL,
  [DeviceName] [varchar](100) NULL,
  [DeviceSerialNo] [varchar](50) NULL,
  [PersonName] [varchar](100) NULL,
  [CardNo] [varchar](50) NULL,
  [sync] [tinyint] NULL,
  [creationTime] [datetime] NULL
) ON [PRIMARY]
GO
ALTER TABLE [dbo].[TabEmployeeAttendance] ADD  DEFAULT ((0)) FOR [sync]
GO
ALTER TABLE [dbo].[TabEmployeeAttendance] ADD  DEFAULT (getdate()) FOR [creationTime]
GO
```
```sql
USE [hikvisiondb]
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[TabEmployeeAttendance](
  [EmployeeID] [varchar](50) NULL,
  [AuthenticationDateAndTime] [varchar](100) NULL,
  [AuthenticationDate] [varchar](100) NULL,
  [AuthenticationTime] [varchar](100) NULL,
  [Direction] [varchar](50) NULL,
  [DeviceName] [varchar](100) NULL,
  [DeviceSerialNo] [varchar](50) NULL,
  [PersonName] [varchar](100) NULL,
  [CardNo] [varchar](50) NULL,
  [sync] [tinyint] NULL,
  [creationTime] [datetime] NULL
) ON [PRIMARY]
GO
ALTER TABLE [dbo].[TabEmployeeAttendance] ADD  DEFAULT ((0)) FOR [sync]
GO
ALTER TABLE [dbo].[TabEmployeeAttendance] ADD  DEFAULT (getdate()) FOR [creationTime]
GO
```
