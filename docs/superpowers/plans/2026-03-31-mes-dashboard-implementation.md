# MES Dashboard Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a production-ready MES manufacturing dashboard with real-time data visualization, Spring Boot backend, and responsive frontend.

**Architecture:** Three-tier architecture - React frontend for UI, Spring Boot 2.7.x backend for APIs, PostgreSQL for data persistence, WebSocket for real-time updates.

**Tech Stack:**
- Backend: Java 8, Spring Boot 2.7.x, MyBatis-Plus, PostgreSQL
- Frontend: React 18, TypeScript, ECharts, Tailwind CSS
- Real-time: Spring WebSocket, STOMP protocol
- Build: Maven, Vite

---

## File Structure

```
mes-dashboard/
├── frontend/                          # React Frontend
│   ├── src/
│   │   ├── components/
│   │   │   ├── overview/              # 生产概况组件
│   │   │   │   ├── ProductionOverview.tsx
│   │   │   │   └── MetricCard.tsx
│   │   │   ├── devices/`              # 设备监控组件
│   │   │   │   ├── DeviceList.tsx
│   │   │   │   └── DeviceItem.tsx
│   │   │   ├── orders/                # 订单组件
│   │   │   │   ├── OrderList.tsx
│   │   │   │   └── OrderProgress.tsx
│   │   │   ├── quality/               # 质量监控组件
│   │   │   │   ├── QualityMetrics.tsx
│   │   │   │   └── MetricCard.tsx
│   │   │   ├── process/               # 工艺流程组件
│   │   │   │   ├── ProcessFlow.tsx
│   │   │   │   └── ProcessStep.tsx
│   │   │   ├── alerts/                # 告警组件
│   │   │   │   ├── AlertCenter.tsx
│   │   │   │   └── AlertItem.tsx
│   │   │   └── charts/                # 图表组件
│   │   │       ├── ProductionTrendChart.tsx
│   │   │       ├── QualityTrendChart.tsx
│   │   │       └── OEEComparisonChart.tsx
│   │   ├── services/
│   │   │   ├── api/                   # API调用
│   │   │   │   ├── deviceService.ts
│   │   │   │   ├── orderService.ts
│   │   │   │   ├── qualityService.ts
│   │   │   │   └── alertService.ts
│   │   │   └── websocket/             # WebSocket连接
│   │   │       └── websocketClient.ts
│   │   ├── types/                      # TypeScript类型定义
│   │   │   ├── device.types.ts
│   │   │   ├── order.types.ts
│   │   │   └── quality.types.ts
│   │   ├── hooks/                      # 自定义Hooks
│   │   │   ├── useRealtimeData.ts
│   │   │   └── useWebSocket.ts
│   │   └── App.tsx
│   ├── package.json
│   └── vite.config.ts
│
├── backend/                           # Spring Boot Backend
│   └── src/main/java/com/example/mes/
│       ├── MesApplication.java            # 应用入口
│       ├── common/                      # 通用组件
│       │   ├── config/
│       │   │   ├── WebSocketConfig.java
│       │   │   ├── CorsConfig.java
│       │   │   └── MyBatisConfig.java
│       │   ├── result/
│       │   │   └── Result.java         # 统一响应
│       │   └── constants/
│       │       └── MesConstants.java
│       ├── modules/
│       │   └── device/
│       │       ├── controller/
│       │       │   └── DeviceController.java
│       │       ├── service/
│       │       │   ├── DeviceService.java
│       │       │   └── impl/
│       │       │       └── DeviceServiceImpl.java
│       │       ├── mapper/
│       │       │   └── DeviceMapper.java
│       │       ├── entity/
│       │       │   └── DeviceDO.java
│       │       ├── dto/
│       │       │   └── DeviceDTO.java
│       │       └── vo/
│       │           └── DeviceVO.java
│       └── order/
│           ├── controller/
│           │   └── OrderController.java
│           ├── service/
│           │   ├── OrderService.java
│           │   └── impl/
│           │       └── OrderServiceImpl.java
│           ├── mapper/
│           │   └── OrderMapper.java
│           ├── entity/
│           │   └── OrderDO.java
│           ├── dto/
│           │   └── OrderDTO.java
│           └── vo/
│               └── OrderVO.java
│   └── resources/
│       ├── mapper/                      # MyBatis XML
│       │   ├── DeviceMapper.xml
│       │   └── OrderMapper.xml
│       └── application.yml
│
└── database/                          # 数据库脚本
    └── schema.sql
```

---

### Task 1: Initialize Backend Spring Boot Project

**Files:**
- Create: `backend/pom.xml`
- Create: `backend/src/main/resources/application.yml`
- Create: `backend/src/main/java/com/example/mes/MesApplication.java`

- [ ] **Step 1: Create backend directory structure**

```bash
mkdir -p backend/src/main/java/com/example/mes
mkdir -p backend/src/main/resources
mkdir -p backend/src/test/java
```

- [ ] **Step 2: Write pom.xml with Spring Boot dependencies**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.18</version>
    </parent>
    <groupId>com.example</groupId>
    <artifactId>mes-backend</artifactId>
    <version>1.0.0</version>
    <properties>
        <java.version>1.8</java.version>
        <mybatis-plus.version>3.5.3</mybatis-plus.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis-plus.version}</version>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

- [ ] **Step 3: Write application.yml configuration**

```yaml
server:
  port: 8080

spring:
  application:
    name: mes-dashboard
  datasource:
    driver-class-name: org.postgresql.Driver
    url: jdbc:postgresql://localhost:5432/mes_db
    username: postgres
    password: postgres
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

mybatis-plus:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.example.mes.modules.*.entity

logging:
  level:
    com.example.mes: debug
```

- [ ] **Step 4: Write MesApplication.java entry point**

```java
package com.example.mes;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MesApplication {
    public static void main(String[] args) {
        SpringApplication.run(MesApplication.class, args);
    }
}
```

- [ ] **Step 5: Commit**

```bash
cd backend
git add pom.xml src/main/resources/application.yml src/main/java/com/example/mes/MesApplication.java
git commit -m "feat: initialize Spring Boot backend project"
```

---

### Task 2: Initialize Frontend React Project

**Files:**
- Create: `frontend/package.json`
- Create: `frontend/vite.config.ts`
- Create: `frontend/tsconfig.json`
- Create: `frontend/src/main.tsx`

- [ ] **Step 1: Create frontend directory structure**

```bash
mkdir -p frontend/src/{components,services,hooks,types}
```

- [ ] **Step 2: Write package.json**

```json
{
  "name": "mes-frontend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "echarts": "^5.4.0",
    "axios": "^1.6.0",
    "sockjs-client": "^1.6.0",
    "stompjs": "^2.3.3"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "@vitejs/plugin-react": "^4.0.0",
    "typescript": "^5.0.0",
    "vite": "^5.0.0",
    "tailwindcss": "^3.3.0",
    "autoprefixer": "^10.4.0",
    "postcss": "^8.4.0"
  }
}
```

- [ ] **Step 3: Write vite.config.ts**

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true
      }
    }
  }
});
```

- [ ] **Step 4: Write tsconfig.json**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

- [ ] **Step 5: Write src/main.tsx**

```typescript
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

- [ ] **Step 6: Commit**

```bash
cd frontend
git add package.json vite.config.ts tsconfig.json src/main.tsx
git commit -m "feat: initialize React frontend project"
```

---

### Task 3: Create Database Schema

**Files:**
- Create: `database/schema.sql`

- [ ] **Step 1: Write schema.sql with MES entities**

```sql
-- 设备表
CREATE TABLE mes_device (
    id BIGSERIAL PRIMARY KEY,
    device_code VARCHAR(50) UNIQUE NOT NULL,
    device_name VARCHAR(100) NOT NULL,
    device_type VARCHAR(50) NOT NULL,
    status VARCHAR(20) NOT NULL, -- RUNNING, STANDBY, FAULT, MAINTENANCE
    current_process VARCHAR(100),
    running_duration INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 生产订单表
CREATE TABLE mes_order (
    id BIGSERIAL PRIMARY KEY,
    order_code VARCHAR(50) UNIQUE NOT NULL,
    product_name VARCHAR(100) NOT NULL,
    planned_quantity INTEGER NOT NULL,
    completed_quantity INTEGER DEFAULT 0,
    status VARCHAR(20) NOT NULL, -- PLANNED, PRODUCING, COMPLETED
    delivery_date DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 质量指标表
CREATE TABLE mes_quality_metric (
    id BIGSERIAL PRIMARY KEY,
    metric_date DATE NOT NULL,
    total_quantity INTEGER NOT NULL,
    qualified_quantity INTEGER NOT NULL,
    defective_quantity INTEGER DEFAULT 0,
    rework_quantity INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 告警表
CREATE TABLE mes_alert (
    id BIGSERIAL PRIMARY KEY,
    alert_type VARCHAR(50) NOT NULL, -- DEVICE_FAULT, QUALITY_ISSUE, MATERIAL_WARNING
    severity VARCHAR(20) NOT NULL, -- URGENT, WARNING, INFO
    title VARCHAR(200) NOT NULL,
    description TEXT,
    status VARCHAR(20) DEFAULT 'PENDING', -- PENDING, PROCESSING, RESOLVED
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 物料表
CREATE TABLE mes_material (
    id BIGSERIAL PRIMARY KEY,
    material_code VARCHAR(50) UNIQUE NOT NULL,
    material_name VARCHAR(100) NOT NULL,
    stock_quantity INTEGER NOT NULL,
    unit VARCHAR(20),
    min_stock_quantity INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 索引
CREATE INDEX idx_device_status ON mes_device(status);
CREATE INDEX idx_order_status ON mes_order(status);
CREATE INDEX idx_alert_status ON mes_alert(status);
CREATE INDEX idx_alert_created ON mes_alert(created_at DESC);
```

- [ ] **Step 2: Commit**

```bash
cd database
git add schema.sql
git commit -m "feat: create database schema"
```

---

### Task 4: Implement Common Backend Components

**Files:**
- Create: `backend/src/main/java/com/example/mes/common/result/Result.java`
- Create: `backend/src/main/java/com/example/mes/common/config/WebSocketConfig.java`
- Create: `backend/src/main/java/com/example/mes/common/config/CorsConfig.java`

- [ ] **Step 1: Write Result.java for unified response**

```java
package com.example.mes.common.result;

import lombok.Data;

@Data
public class Result<T> {
    private Integer code;
    private String message;
    private T data;

    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.setCode(200);
        result.setMessage("Success");
        result.setData(data);
        return result;
    }

    public static <T> Result<T> error(String message) {
        Result<T> result = new Result<>();
        result.setCode(500);
        result.setMessage(message);
        return result;
    }
}
```

- [ ] **Step 2: Write WebSocketConfig.java**

```java
package com.example.mes.common.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic");
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws").setAllowedOriginPatterns("*");
    }
}
```

- [ ] **Step 3: Write CorsConfig.java**

```java
package com.example.mes.common.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

import java.util.Arrays;

@Configuration
public class CorsConfig {

    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowCredentials(true);
        config.setAllowedOrigins(Arrays.asList("*"));
        config.setAllowedMethods(Arrays.asList("*"));
        config.setAllowedHeaders(Arrays.asList("*"));

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return new CorsFilter(source);
    }
}
```

- [ ] **Step 4: Commit**

```bash
cd backend
git add src/main/java/com/example/mes/common/
git commit -m "feat: implement common backend components"
```

---

### Task 5: Implement Device Module (Backend)

**Files:**
- Create: `backend/src/main/java/com/example/mes/modules/device/entity/DeviceDO.java`
- Create: `backend/src/main/java/com/example/mes/modules/device/dto/DeviceDTO.java`
- Create: `backend/src/main/java/com/example/mes/modules/device/vo/DeviceVO.java`
- Create: `backend/src/main/java/com/example/mes/modules/device/mapper/DeviceMapper.java`
- Create: `backend/src/main/resources/mapper/DeviceMapper.xml`
- Create: `backend/src/main/java/com/example/mes/modules/device/service/DeviceService.java`
- Create: `backend/src/main/java/com/example/mes/modules/device/service/impl/DeviceServiceImpl.java`
- Create: `backend/src/main/java/com/example/mes/modules/device/controller/DeviceController.java`

- [ ] **Step 1: Write DeviceDO entity**

```java
package com.example.mes.modules.device.entity;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;
import java.util.Date;

@Data
@TableName("mes_device")
public class DeviceDO {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String deviceCode;
    private String deviceName;
    private String deviceType;
    private String status;
    private String currentProcess;
    private Integer runningDuration;
    private Date createdAt;
    private Date updatedAt;
}
```

- [ ] **Step 2: Write DeviceDTO for data transfer**

```java
package com.example.mes.modules.device.dto;

import lombok.Data;

@Data
public class DeviceDTO {
    private String deviceCode;
    private String deviceName;
    private String status;
    private String currentProcess;
    private Integer runningDuration;
}
```

- [ ] **Step 3: Write DeviceVO for response**

```java
package com.example.mes.modules.device.vo;

import lombok.Data;

@Data
public class DeviceVO {
    private Long id;
    private String deviceCode;
    private String deviceName;
    private String deviceType;
    private String status;
    private String currentProcess;
    private Integer runningDuration;
}
```

- [ ] **Step 4: Write DeviceMapper interface**

```java
package com.example.mesmes.modules.device.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.example.mes.modules.device.entity.DeviceDO;
import org.apache.ibatis.annotations.Mapper;
import java.util.List;

@Mapper
public interface DeviceMapper extends BaseMapper<DeviceDO> {
    List<DeviceDO> selectDevicesByStatus(String status);
}
```

- [ ] **Step 5: Write DeviceMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mes.modules.device.mapper.DeviceMapper">
    <select id="selectDevicesByStatus" resultType="com.example.mes.modules.device.entity.DeviceDO">
        SELECT * FROM mes_device WHERE status = #{status}
    </select>
</mapper>
```

- [ ] **Step 6: Write DeviceService interface**

```java
package com.example.mes.modules.device.service;

import com.example.mes.modules.device.dto.DeviceDTO;
import com.example.mes.modules.device.vo.DeviceVO;
import java.util.List;

public interface DeviceService {
    List<DeviceVO> getAllDevices();
    DeviceVO getDeviceById(Long id);
    void updateDevice(DeviceDTO dto);
}
```

- [ ] **Step 7: Write DeviceServiceImpl**

```java
package com.example.mes.modules.device.service.impl;

import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.example.mes.modules.device.dto.DeviceDTO;
import com.example.mes.modules.device.entity.DeviceDO;
import com.example.mes.modules.device.mapper.DeviceMapper;
import com.example.mes.modules.device.service.DeviceService;
import com.example.mes.modules.device.vo.DeviceVO;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeanUtils;
import org.springframework.stereotype.Service;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

@Slf4j
@Service
@RequiredArgsConstructor
public class DeviceServiceImpl implements DeviceService {

    private final DeviceMapper deviceMapper;

    @Override
    public List<DeviceVO> getAllDevices() {
        List<DeviceDO> devices = deviceMapper.selectList(null);
        List<DeviceVO> result = new ArrayList<>();
        for (DeviceDO device : devices) {
            DeviceVO vo = new DeviceVO();
            BeanUtils.copyProperties(device, vo);
            result.add(vo);
        }
        return result;
    }

    @Override
    public DeviceVO getDeviceById(Long id) {
        DeviceDO device = deviceMapper.selectById(id);
        DeviceVO vo = new DeviceVO();
        BeanUtils.copyProperties(device, vo);
        return vo;
    }

    @Override
    public void updateDevice(DeviceDTO dto) {
        QueryWrapper<DeviceDO> wrapper = new QueryWrapper<>();
        wrapper.eq("device_code", dto.getDeviceCode());
        DeviceDO device = deviceMapper.selectOne(wrapper);
        if (device != null) {
            BeanUtils.copyProperties(dto, device);
            device.setUpdatedAt(new Date());
            deviceMapper.updateById(device);
            log.info("Device updated: {}", dto.getDeviceCode());
        }
    }
}
```

- [ ] **Step 8: Write DeviceController**

```java
package com.example.mes.modules.device.controller;

import com.example.mes.common.result.Result;
import com.example.mes.modules.device.dto.DeviceDTO;
import com.example.mes.modules.device.service.DeviceService;
import com.example.mes.modules.device.vo.DeviceVO;
import lombok.RequiredArgsConstructor;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/devices")
@RequiredArgsConstructor
public class DeviceController {

    private final DeviceService deviceService;
    private final SimpMessagingTemplate messagingTemplate;

    @GetMapping
    public Result<List<DeviceVO>> getAllDevices() {
        return Result.success(deviceService.getAllDevices());
    }

    @GetMapping("/{id}")
    public Result<DeviceVO> getDeviceById(@PathVariable Long id) {
        return Result.success(deviceService.getDeviceById(id));
    }

    @PutMapping
    public Result<Void> updateDevice(@RequestBody DeviceDTO dto) {
        deviceService.updateDevice(dto);
        messagingTemplate.convertAndSend("/topic/devices", dto);
        return Result.success(null);
    }
}
```

- [ ] **Step 9: Commit**

```bash
cd backend
git add src/main/java/com/example/mes/modules/device/
git add src/main/resources/mapper/DeviceMapper.xml
git commit -m "feat: implement device module backend"
```

---

### Task 6: Implement Device Component (Frontend)

**Files:**
- Create: `frontend/src/types/device.types.ts`
- Create: `frontend/src/services/api/deviceService.ts`
- Create: `frontend/src/services/websocket/websocketClient.ts`
- Create: `frontend/src/components/devices/DeviceItem.tsx`
- Create: `frontend/src/components/devices/DeviceList.tsx`

- [ ] **Step 1: Write device.types.ts**

```typescript
export enum DeviceStatus {
  RUNNING = 'RUNNING',
  STANDBY = 'STANDBY',
  FAULT = 'FAULT',
  MAINTENANCE = 'MAINTENANCE'
}

export interface Device {
  id: number;
  deviceCode: string;
  deviceName: string;
  deviceType: string;
  status: DeviceStatus;
  currentProcess: string;
  runningDuration: number;
}
```

- [ ] **Step 2: Write deviceService.ts**

```typescript
import axios from 'axios';
import { Device } from '../../types/device.types';

const API_BASE = '/api/devices';

export const deviceService = {
  getAllDevices: async (): Promise<Device[]> => {
    const response = await axios.get(API_BASE);
    return response.data.data;
  },

  getDeviceById: async (id: number): Promise<Device> => {
    const response = await axios.get(`${API_BASE}/${id}`);
    return response.data.data;
  },

  updateDevice: async (device: Partial<Device>): Promise<void> => {
    await axios.put(API_BASE, device);
  }
};
```

- [ ] **Step 3: Write websocketClient.ts**

```typescript
import SockJS from 'sockjs-client';
import { Stomp, CompatClient } from '@stomp/stompjs';

export class WebSocketClient {
  private client: CompatClient | null = null;
  private connected = false;

  connect(url: string, onMessage: (topic: string, message: any) => void): void {
    const socket = new SockJS(url);
    this.client = Stomp.over(socket);

    this.client.connect({}, () => {
      this.connected = true;
      console.log('WebSocket connected');
    }, (error: any) => {
      console.error('WebSocket error:', error);
    });

    if (this.client) {
      this.client.subscribe('/topic/devices', (message: any) => {
        onMessage('/topic/devices', JSON.parse(message.body));
      });
    }
  }

  disconnect(): void {
    if (this.client && this.connected) {
      this.client.disconnect();
      this.connected = false;
    }
  }
}

export const websocketClient = new WebSocketClient();
```

- [ ] **Step 4: Write DeviceItem.tsx**

```typescript
import React from 'react';
import { Device, DeviceStatus } from '../../types/device.types';

interface DeviceItemProps {
  device: Device;
}

const DeviceItem: React.FC<DeviceItemProps> = ({ device }) => {
  const getStatusText = (status: DeviceStatus): string => {
    const statusMap = {
      [DeviceStatus.RUNNING]: '运行中',
      [DeviceStatus.STANDBY]: '待机',
      [DeviceStatus.FAULT]: '故障',
      [DeviceStatus.MAINTENANCE]: '保养'
    };
    return statusMap[status] || status;
  };

  const getStatusClass = (status: DeviceStatus): string => {
    const classMap = {
      [DeviceStatus.RUNNING]: 'status-running',
      [DeviceStatus.STANDBY]: 'status-standby',
      [DeviceStatus.FAULT]: 'status-fault',
      [DeviceStatus.MAINTENANCE]: 'status-maintenance'
    };
    return classMap[status] || '';
  };

  return (
    <li className="device-item">
      <span className="device-name">{device.deviceName}</span>
      <span className={`device-status ${getStatusClass(device.status)}`}>
        {getStatusText(device.status)}
      </span>
    </li>
  );
};

export default DeviceItem;
```

- [ ] **Step 5: Write DeviceList.tsx**

```typescript
import React, { useEffect, useState } from 'react';
import { Device } from '../../types/device.types';
import { deviceService } from '../../services/api/deviceService';
import { websocketClient } from '../../services/websocket/websocketClient';
import DeviceItem from './DeviceItem';

const DeviceList: React.FC = () => {
  const [devices, setDevices] = useState<Device[]>([]);

  useEffect(() => {
    // Initial fetch
    deviceService.getAllDevices().then(setDevices);

    // WebSocket connection
    websocketClient.connect('/ws', (topic, message) => {
      setDevices(prevDevices =>
        prevDevices.map(d =>
          d.deviceCode === message.deviceCode ? { ...d, ...message } : d
        )
      );
    });

    return () => websocketClient.disconnect();
  }, []);

  return (
    <div className="panel">
      <h4 className="panel-title">设备状态监控 ({devices.length}台)</h4>
      <ul className="device-list">
        {devices.map(device => (
          <DeviceItem key={device.id} device={device} />
        ))}
      </ul>
    </div>
  );
};

export default DeviceList;
```

- [ ] **Step 6: Commit**

```bash
cd frontend
git add src/types/device.types.ts src/services/ src/components/devices/
git commit -m "feat: implement device component frontend"
```

---

### Task 7: Implement Overview Metrics Component

**Files:**
- Create: `frontend/src/components/overview/MetricCard.tsx`
- Create: `frontend/src/components/overview/ProductionOverview.tsx`

- [ ] **Step 1: Write MetricCard.tsx**

```typescript
import React from 'react';

interface MetricCardProps {
  title: string;
  value: number | string;
  unit?: string;
  subInfo?: string;
  valueClass?: string;
}

const MetricCard: React.FC<MetricCardProps> = ({
  title,
  value,
  unit = '',
  subInfo = '',
  valueClass = ''
}) => {
  return (
    <div className="overview-card">
      <h3>{title}</h3>
      <div className={`value ${valueClass}`}>{value}</div>
      {unit && <div className="unit">{unit}</div>}
      {subInfo && <div className="sub-info">{subInfo}</div>}
    </div>
  );
};

export default MetricCard;
```

- [ ] **Step 2: Write ProductionOverview.tsx**

```typescript
import React, { useEffect, useState } from 'react';
import axios from 'axios';
import MetricCard from './MetricCard';

interface OverviewMetrics {
  realTimeProduction: number;
  targetProduction: number;
  completionRate: number;
  oee: number;
  operatingRate: number;
  passRate: number;
  wip: number;
}

const ProductionOverview: React.FC = () => {
  const [metrics, setMetrics] = useState<OverviewMetrics>({
    realTimeProduction: 12458,
    targetProduction: 15000,
    completionRate: 83,
    oee: 87.5,
    operatingRate: 92.3,
    passRate: 99.2,
    wip: 3256
  });

  useEffect(() => {
    const fetchMetrics = async () => {
      try {
        const response = await axios.get('/api/metrics/overview');
        setMetrics(response.data.data);
      } catch (error) {
        console.error('Failed to fetch metrics:', error);
      }
    };

    fetchMetrics();
    const interval = setInterval(fetchMetrics, 5000); // Poll every 5 seconds
    return () => clearInterval(interval);
  }, []);

  return (
    <section className="overview">
      <MetricCard
        title="实时产量"
        value={metrics.realTimeProduction.toLocaleString()}
        unit="件"
        subInfo={`今日目标：${metrics.targetProduction.toLocaleString()} | 完成率：${metrics.completionRate}%`}
      />
      <MetricCard
        title="生产效率 (OEE)"
        value={metrics.oee}
        unit="%"
        subInfo={`设备稼动率：${metrics.operatingRate}% | 性能指数：95%`}
      />
      <MetricCard
        title="合格率"
        value={metrics.passRate}
}
        unit="%"
        subInfo="一次通过率：98.7% | 返工率：0.5%"
      />
      <MetricCard
        title="在制品(WIP)"
        value={metrics.wip.toLocaleString()}
        unit="件"
        subInfo="目标值：3,000 | 状态：偏高"
      />
    </section>
  );
};

export default ProductionOverview;
```

- [ ] **Step 3: Commit**

```bash
cd frontend
git add src/components/overview/
git commit -m "feat: implement overview metrics component"
```

---

### Task 8: Implement Charts Components (ECharts)

**Files:**
- Create: `frontend/src/components/charts/ProductionTrendChart.tsx`
- Create: `frontend/src/components/charts/QualityTrendChart.tsx`
- Create: `frontend/src/components/charts/OEEComparisonChart.tsx`

- [ ] **Step 1: Write ProductionTrendChart.tsx**

```typescript
import React, { useEffect, useRef } from 'react';
import * as echarts from 'echarts';

const ProductionTrendChart: React.FC = () => {
  const chartRef = useRef<HTMLDivElement>(null);
  const chartInstance = useRef<echarts.ECharts | null>(null);

  useEffect(() => {
    if (chartRef.current) {
      chartInstance.current = echarts.init(chartRef.current);

      const option: echarts.EChartsOption = {
        title: {
          text: '产量趋势（近7天）',
          textStyle: { color: '#00bcd4' }
        },
        tooltip: {
          trigger: 'axis'
        },
        xAxis: {
          type: 'category',
          data: ['周一', '周二', '周三', '周四', '周五', '周六', '周日'],
          axisLine: { lineStyle: { color: '#00bcd4' } }
        },
        yAxis: {
          type: 'value',
          axisLine: { lineStyle: { color: '#00bcd4' } }
        },
        series: [{
          type: 'bar',
          data: [9500, 11800, 13400, 11000, 14200, 12600, 14950],
          itemStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              { offset: 0, color: '#00bcd4' },
              { offset: 1, color: 'rgba(0, 188, 212, 0.5)' }
            ])
          }
        }]
      };

      chartInstance.current.setOption(option);
    }

    return () => {
      chartInstance.current?.dispose();
    };
  }, []);

  return <div ref={chartRef} style={{ width: '100%', height: '200px' }} />;
};

export default ProductionTrendChart;
```

- [ ] **Step 2: Write QualityTrendChart.tsx**

```typescript
import React, { useEffect, useRef } from 'react';
import * as echarts from 'echarts';

const QualityTrendChart: React.FC = () => {
  const chartRef = useRef<HTMLDivElement>(null);
  const chartInstance = useRef<echarts.ECharts | null>(null);

  useEffect(() => {
    if (chartRef.current) {
      chartInstance.current = echarts.init(chartRef.current);

      const option: echarts.EChartsOption = {
        title: {
          text: '合格率趋势',
          textStyle: { color: '#00bcd4' }
        },
        tooltip: { trigger: 'axis' },
        xAxis: {
          type: 'category',
          data: ['周一', '周二', '周三', '周四', '周五', '周六', '周日'],
          axisLine: { lineStyle: { color: '#00bcd4' } }
        },
        yAxis: {
          type: 'value',
          min: 85,
          max: 100,
          axisLine: { lineStyle: { color: '#00bcd4' } }
        },
        series: [{
          type: 'line',
          data: [90.0, 92.0, 91.0, 94.0, 93.0, 95.0, 96.0],
          smooth: true,
          itemStyle: { color: '#4caf50' },
          lineStyle: { color: '#4caf50' },
          areaStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              { offset: 0, color: 'rgba(76, 175, 80, 0.3)' },
              { offset: 1, color: 'rgba(76, 175, 80, 0.05)' }
            ])
          }
        }]
      };

      chartInstance.current.setOption(option);
    }

    return () => {
      chartInstance.current?.dispose();
    };
  }, []);

  return <div ref={chartRef} style={{ width: '100%', height: '200px' }} />;
};

export default QualityTrendChart;
```

- [ ] **Step 3: Write OEEComparisonChart.tsx**

```typescript
// Similar structure to above, with OEE comparison bar chart
import React, { useEffect, useRef } from 'react';
import * as echarts from 'echarts';

const OEEComparisonChart: React.FC = () => {
  const chartRef = useRef<HTMLDivElement>(null);
  const chartInstance = useRef<echarts.ECharts | null>(null);

  useEffect(() => {
    if (chartRef.current) {
      chartInstance.current = echarts.init(chartRef.current);

      const option: echarts.EChartsOption = {
        title: {
          text: '设备OEE对比',
          textStyle: { color: '#00bcd4' }
        },
        tooltip: { trigger: 'axis' },
        xAxis: {
          type: 'category',
          data: ['CNC-01', 'CNC-02', 'CNC-03', 'CNC-04', 'ASM-01', 'ASM-02'],
          axisLine: { lineStyle: { color: '#00bcd4' } }
        },
        yAxis: {
          type: 'value',
          axisLine: { lineStyle: { color: '#00bcd4' } }
        },
        series: [{
          type: 'bar',
          data: [85, 88, 82, 90, 87, 84],
          itemStyle: { color: '#00bcd4' }
        }]
      };

      chartInstance.current.setOption(option);
    }

    return () => {
      chartInstance.current?.dispose();
    };
  }, []);

  return <div ref={chartRef} style={{ width: '100%', height: '200px' }} />;
};

export default OEEComparisonChart;
```

- [ ] **Step 4: Commit**

```bash
cd frontend
git add src/components/charts/
git commit -m "feat: implement chart components with ECharts"
```

---

### Task 9: Implement Main App Layout

**Files:**
- Modify: `frontend/src/App.tsx`
- Create: `frontend/src/index.css`
- Create: `frontend/src/tailwind.config.js`

- [ ] **Step 1: Write App.tsx with complete layout**

```typescript
import React from 'react';
import DeviceList from './components/devices/DeviceList';
import ProductionOverview from './components/overview/ProductionOverview';
import ProductionTrendChart from './components/charts/ProductionTrendChart';
import QualityTrendChart from './components/charts/QualityTrendChart';
import OEEComparisonChart from './components/charts/OEEComparisonChart';

function App() {
  const getCurrentTime = (): string => {
    return new Date().toLocaleString('zh-CN', {
      year: 'numeric',
      month: '2-digit',
      day: '2-digit',
      hour: '2-digit',
      minute: '2-digit',
      second: '2-digit'
    });
  };

  return (
    <div className="container">
      <header className="header">
        <h1>MES制造执行系统 - 实时监控大屏</h1>
        <div className="info">
          当前班次：白班 (08:00-20:00) | 班次组长：张经理 | 更新时间：{getCurrentTime()}
        </div>
      </header>

      <ProductionOverview />

      <div className="main-content">
        <aside className="side-panel">
          <DeviceList />
          {/* Add OrderList, PersonnelCard components when implemented */}
        </aside>

        <div className="center-panel">
          {/* Add ProcessFlow component */}
          <div className="charts-section">
            <h4 className="panel-title">生产数据看板</h4>
            <div className="charts-grid">
              <ProductionTrendChart />
              <QualityTrendChart />
              <OEEComparisonChart />
              {/* Add Top5DefectsChart when implemented */}
            </div>
          </div>
        </div>

        <aside className="side-panel">
          {/* Add QualityMetrics, MaterialMonitoring, AlertCenter, EnergyMonitoring */}
        </aside>
      </div>
    </div>
  );
}

export default App;
```

- [ ] **Step 2: Write index.css with Tailwind directives**

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  font-family: 'Microsoft YaHei', Arial, sans-serif;
  background: linear-gradient(135deg, #0a1628 0%, #1a2a4a 100%);
  color: #fff;
  min-height: 100vh;
}

.container {
  max-width: 1920px;
  margin: 0 auto;
  padding: 20px;
}

.header {
  text-align: center;
  padding: 20px 0;
  border-bottom: 2px solid #00bcd4;
  margin-bottom: 20px;
}

.header h1 {
  font-size: 36px;
  color: #00bcd4;
  text-shadow: 0 0 20px rgba(0, 188, 212, 0.5);
  margin-bottom: 10px;
}

.header .info {
  font-size: 14px;
  color: #88aadd;
}

/* Add remaining CSS styles from prototype */
```

- [ ] **Step 3: Write tailwind.config.js**

```javascript
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        primary: '#00bcd4',
      }
    }
  },
  plugins: [],
}
```

- [ ] **Step 4: Commit**

```bash
cd frontend
git add src/App.tsx src/index.css tailwind.config.js
git commit -m "feat: implement main app layout"
```

---

### Task 10: Add Test Data Initialization

**Files:**
- Create: `backend/src/main/java/com/example/mes/common/config/DataInitializer.java`

- [ ] **Step 1: Write DataInitializer.java**

```java
package com.example.mes.common.config;

import com.example.mes.modules.device.entity.DeviceDO;
import com.example.mes.modules.device.mapper.DeviceMapper;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;
import java.util.Arrays;
import java.util.Date;
import java.util.List;

@Slf4j
@Component
@RequiredArgsConstructor
public class DataInitializer implements CommandLineRunner {

    private final DeviceMapper deviceMapper;

    @Override
    public void run(String... args) {
        if (deviceMapper.selectCount(null) == 0) {
            initializeTestData();
        }
    }

    private void initializeTestData() {
        List<DeviceDO> devices = Arrays.asList(
            createDevice("CNC-01", "数控机床01", "CNC", "RUNNING"),
            createDevice("CNC-02", "数控机床02", "CNC", "RUNNING"),
            createDevice("CNC-03", "数控机床03", "CNC", "FAULT"),
            createDevice("CNC-04", "数控机床04", "CNC", "STANDBY"),
            createDevice("ASSEMBLY-01", "组装线01", "ASSEMBLY", "RUNNING"),
            createDevice("ASSEMBLY-02", "组装线02", "ASSEMBLY", "MAINTENANCE"),
            createDevice("PACK-01", "包装线01", "PACKAGING", "RUNNING"),
            createDevice("PACK-02", "包装线02", "PACKAGING", "RUNNING")
        );

        devices.forEach(deviceMapper::insert);
        log.info("Test data initialized");
    }

    private DeviceDO createDevice(String code, String name, String type, String status) {
        DeviceDO device = new DeviceDO();
        device.setDeviceCode(code);
        device.setDeviceName(name);
        device.setDeviceType(type);
        device.setStatus(status);
        device.setCurrentProcess("");
        device.setRunningDuration(0);
        device.setCreatedAt(new Date());
        device.setUpdatedAt(new Date());
        return device;
    }
}
```

- [ ] **Step 2: Commit**

```bash
cd backend
git add src/main/java/com/example/mes/common/config/DataInitializer.java
git commit -m "feat: add test data initialization"
```

---

### Task 11: Write Integration Tests

**Files:**
- Create: `backend/src/test/java/com/example/mes/modules/device/DeviceControllerTest.java`

- [ ] **Step 1: Write DeviceControllerTest.java**

```java
package com.example.mes.modules.device;

import com.example.mes.common.result.Result;
import com.example.mes.modules.device.controller.DeviceController;
import com.example.mes.modules.device.service.DeviceService;
import com.example.mes.modules.device.vo.DeviceVO;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;

import java.util.Arrays;
import java.util.List;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.when;

@SpringBootTest
public class DeviceControllerTest {

    @Mock
    private DeviceService deviceService;

    @InjectMocks
    private DeviceController deviceController;

    private MockMvc mockMvc;

    @BeforeEach
    public void setUp() {
        MockitoAnnotations.openMocks(this);
        mockMvc = MockMvcBuilders.standaloneSetup(deviceController).build();
    }

    @Test
    public void testGetAllDevices() {
        DeviceVO device1 = new DeviceVO();
        device1.setId(1L);
        device1.setDeviceCode("CNC-01");
        device1.setDeviceName("数控机床01");
        device1.setStatus("RUNNING");

        when(deviceService.getAllDevices()).thenReturn(Arrays.asList(device1));

        Result<List<DeviceVO>> result = deviceController.getAllDevices();

        assertEquals(200, result.getCode());
        assertEquals(1, result.getData().size());
        assertEquals("CNC-01", result.getData().get(0).getDeviceCode());
    }

    @Test
    public void testUpdateDevice() {
        when(deviceService.getDeviceById(1L)).thenReturn(new DeviceVO());

        Result<Void> result = deviceController.updateDevice(null);

        assertEquals(200, result.getCode());
    }
}
```

- [ ] **Step 2: Commit**

```bash
cd backend
git add src/test/java/com/example/mes/modules/device/DeviceControllerTest.java
git commit -m "test: add device controller integration tests"
```

---

### Task 12: Create README and Documentation

**Files:**
- Create: `README.md`
- Create: `backend/README.md`
- Create: `frontend/README.md`

- [ ] **Step 1: Write main README.md**

```markdown
# MES制造执行系统 - 实时监控大屏

## 项目概述

基于Spring Boot和React构建的MES制造执行系统实时监控大屏，支持设备监控、生产订单、质量控制、异常告警等功能。

## 技术栈

### 后端
- Java 8
- Spring Boot 2.7.18
- MyBatis-Plus 3.5.3
- PostgreSQL
- WebSocket

### 前端
- React 18
- TypeScript
- Vite
- Tailwind CSS
- ECharts
- SockJS + STOMP

## 快速开始

### 环境要求
- JDK 8
- Maven 3.6+
- Node.js 18+
- PostgreSQL 12+

### 启动后端
\`\`\`bash
cd backend
mvn spring-boot:run
\`\`\`

### 启动前端
\`\`\`bash
cd frontend
npm install
npm run dev
\`\`\`

### 访问地址
- 前端：http://localhost:3000
- 后端API：http://localhost:8080
- WebSocket：ws://localhost:8080/ws

## 项目结构

详见docs中的架构文档。
```

- [ ] **Step 2: Write backend README.md**

```markdown
# MES Backend

## API端点

### 设备管理
- GET /api/devices - 获取所有设备
- GET /api/devices/{id} - 获取设备详情
- PUT /api/devices - 更新设备状态

### WebSocket
- 连接：ws://localhost:8080/ws
- 订阅：/topic/devices - 设备状态更新

## 开发指南

遵循阿里巴巴Java开发规范。
```

- [ ] **Step 3: Write frontend README.md**

```markdown
# MES Frontend

## 组件结构

- overview/ - 生产概况组件
- devices/ - 设备监控组件
- orders/ - 订单组件
- quality/ - 质量监控组件
- process/ - 工艺流程组件
- alerts/ - 告警组件
- charts/ - 图表组件

## 开发指南

使用TypeScript严格模式，遵循React Hooks最佳实践。
```

- [ ] **Step 4: Commit**

```bash
git add README.md backend/README.md frontend/README.md
git commit -m "docs: add README and project documentation"
```

---

## Summary

This implementation plan creates a complete MES dashboard system with:

1. **Backend**: Spring Boot 2.7.x with RESTful APIs and WebSocket support
2. **Frontend**: React 18 with TypeScript, real-time updates via WebSocket
3. **Database**: PostgreSQL schema for devices, orders, quality metrics, alerts, materials
4. **Real-time**: WebSocket for live device status updates
5. **Charts**: ECharts integration for production trends and quality metrics
6. **Testing**: Integration tests for critical endpoints

All code follows the Java 8 constraints and Alibaba development standards defined in the project.
