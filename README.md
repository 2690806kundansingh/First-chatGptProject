# Spring AI Setup Guide: OpenAI & Ollama

A complete, beginner-friendly, and working guide to integrating **Spring Boot** with **Spring AI** using both **OpenAI (Cloud LLM)** and **Ollama (Free Local LLM)**.

---

## 📌 Table of Contents
1. [Overview](#-overview)
2. [Prerequisites](#-prerequisites)
3. [Module 1: Spring AI Setup with OpenAI (Cloud LLM)](#-module-1-spring-ai-setup-with-openai-cloud-llm)
   - [Step 1: Generate an OpenAI API Key](#step-1-generate-an-openai-api-key)
   - [Step 2: Create a Spring Boot Project](#step-2-create-a-spring-boot-project)
   - [Step 3: Add Dependencies in `pom.xml`](#step-3-add-dependencies-in-pomxml)
   - [Step 4: Configure API Key in `application.yml`](#step-4-configure-api-key-in-applicationyml)
   - [Step 5: Autowire `ChatClient.Builder` & Create ChatController](#step-5-autowire-chatclientbuilder--create-chatcontroller)
   - [Running and Testing OpenAI](#running-and-testing-openai)
4. [Module 2: Spring AI with Ollama (Free Local LLM)](#-module-2-spring-ai-with-ollama-free-local-llm)
   - [Step 1: Install and Run Ollama](#step-1-install-and-run-ollama)
   - [Step 2: Create a Spring Boot Project for Ollama](#step-2-create-a-spring-boot-project-for-ollama)
   - [Step 3: Add Dependency: Spring AI Ollama Starter](#step-3-add-dependency-spring-ai-ollama-starter)
   - [Step 4: Configure Spring AI for Ollama in `application.yml`](#step-4-configure-spring-ai-for-ollama-in-applicationyml)
   - [Step 5: Autowire `ChatClient.Builder` & Implement Controller](#step-5-autowire-chatclientbuilder--implement-controller)
   - [Running and Testing Ollama](#running-and-testing-ollama)
5. [OpenAI vs Ollama: Comparison](#-openai-vs-ollama-comparison)
6. [Project Structure](#-project-structure)
7. [Deploying / Pushing to GitHub](#-deploying--pushing-to-github)

---

## 📖 Overview
**Spring AI** provides a unified, portable abstraction for interacting with AI models. You write your business logic once using `ChatClient`, and you can effortlessly switch between cloud providers (like **OpenAI**) and local, open-source models (like **Ollama**) with zero changes to your core prompt code!

---

## 🛠 Prerequisites
- **Java 21** or later
- **Maven 3.8+** (or use the included `./mvnw` wrapper)
- Your favorite IDE (IntelliJ IDEA, VS Code, Eclipse)
- For OpenAI: Active OpenAI account with API credits
- For Ollama: Modern PC / laptop (8GB+ RAM recommended)

---

# ☁️ Module 1: Spring AI Setup with OpenAI (Cloud LLM)

---

### Step 1: Generate an OpenAI API Key

To communicate with OpenAI's models (GPT-4o, GPT-3.5-turbo), generate an API secret key:

1. Go to the [OpenAI Platform](https://platform.openai.com/).
2. Log in or create an account.
3. In the left navigation, click **API Keys** (or visit [https://platform.openai.com/api-keys](https://platform.openai.com/api-keys)).
4. Click **+ Create new secret key**.
5. Give your key a name (e.g., `SpringAiKey`) and click **Create secret key**.
6. **Copy the key immediately** and store it safely (OpenAI will not show it again).

#### 🔒 Best Practice: Set as Environment Variable
Never hardcode your API key into code or commit it to GitHub. Instead, set it as an environment variable:

- **Windows (PowerShell)**:
  ```powershell
  # Permanent for user
  [System.Environment]::SetEnvironmentVariable('OPENAI_API_KEY', 'sk-proj-xxxxxxxxxxxxxxxxxxxxxxxx', 'User')

  # For current terminal session
  $env:OPENAI_API_KEY="sk-proj-xxxxxxxxxxxxxxxxxxxxxxxx"
  ```

- **Windows (Command Prompt)**:
  ```cmd
  setx OPENAI_API_KEY "sk-proj-xxxxxxxxxxxxxxxxxxxxxxxx"
  ```

- **macOS / Linux**:
  ```bash
  export OPENAI_API_KEY="sk-proj-xxxxxxxxxxxxxxxxxxxxxxxx"
  # Add to ~/.bashrc or ~/.zshrc for persistence
  ```

---

### Step 2: Create a Spring Boot Project

1. Open [start.spring.io](https://start.spring.io/).
2. Select configuration:
   - **Project**: Maven
   - **Language**: Java
   - **Spring Boot**: 3.x / 4.x
   - **Group**: `com.spring.ai.firstproject`
   - **Artifact**: `first-project`
   - **Name**: `first-project`
   - **Packaging**: Jar
   - **Java**: 21
3. In **Dependencies**, add:
   - **Spring Web** (`spring-boot-starter-web` or `spring-boot-starter-webmvc`)
   - **OpenAI** (`spring-ai-starter-model-openai`)
   - **Lombok** (optional)
4. Click **Generate** and extract the project.

---

### Step 3: Add Dependencies in `pom.xml`

```xml
<properties>
    <java.version>21</java.version>
    <spring-ai.version>2.0.1</spring-ai.version>
</properties>

<dependencies>
    <!-- Spring Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webmvc</artifactId>
    </dependency>

    <!-- Spring AI OpenAI Starter -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-model-openai</artifactId>
    </dependency>

    <!-- Lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

    <!-- Test Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webmvc-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <!-- Spring AI Bill of Materials (BOM) -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>${spring-ai.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

---

### Step 4: Configure API Key in `application.yml`

Create or update `src/main/resources/application.yml`:

```yaml
spring:
  application:
    name: first-project
  ai:
    openai:
      # Reads the key securely from your system environment variable
      api-key: ${OPENAI_API_KEY}

server:
  port: 8081
```

---

### Step 5: Autowire `ChatClient.Builder` & Create ChatController

Create `src/main/java/com/spring/ai/firstproject/first_project/controller/ChatController.java`:

```java
package com.spring.ai.firstproject.first_project.controller;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping
public class ChatController {

    private final ChatClient chatClient;

    // Autowiring ChatClient.Builder via constructor injection
    @Autowired
    public ChatController(ChatClient.Builder builder) {
        this.chatClient = builder.build();
    }

    @GetMapping("/chat")
    public ResponseEntity<String> chat(@RequestParam(value = "q", required = true) String q) {
        var resultResponse = chatClient.prompt(q).call().content();
        return ResponseEntity.ok(resultResponse);
    }
}
```

---

### Running and Testing OpenAI

```bash
# Run application
.\mvnw.cmd spring-boot:run

# Test via Browser or cURL
curl "http://localhost:8081/chat?q=What%20is%20Spring%20AI"
```

---

# 🦙 Module 2: Spring AI with Ollama (Free Local LLM)

Want to run AI completely **free**, **offline**, without any **API keys**, credit cards, or external network requests? **Ollama** lets you run open-source LLMs (Llama 3, Mistral, Phi-3, DeepSeek) locally on your own machine.

---

### Step 1: Install and Run Ollama

1. **Download Ollama**:
   - Visit the official Ollama website: [https://ollama.com/download](https://ollama.com/download)
   - Download the installer for your operating system (**Windows**, **macOS**, or **Linux**) and complete the installation.

2. **Verify Installation in Terminal**:
   Open PowerShell, Command Prompt, or Terminal:
   ```bash
   ollama --version
   ```

3. **Pull and Run a Free Model**:
   Choose an open-source model and run it:
   ```bash
   # Meta's Llama 3 model (recommended)
   ollama run llama3

   # OR Mistral model
   ollama run mistral

   # OR DeepSeek R1 model
   ollama run deepseek-r1:7b

   # OR Microsoft Phi-3 (lightweight, runs fast on any PC)
   ollama run phi3
   ```
   *The first time you run this command, Ollama downloads the model weights locally.*

4. **Verify Ollama Server is Running**:
   Ollama automatically runs a local API server on port `11434`:
   ```bash
   curl http://localhost:11434
   # Output: Ollama is running
   ```

5. **Useful Ollama Commands**:
   - `ollama list` — View all downloaded models on your machine.
   - `ollama ps` — View currently running models.
   - `ollama rm <model-name>` — Delete a model to free disk space.

---

### Step 2: Create a Spring Boot Project for Ollama

1. Navigate to [Spring Initializr](https://start.spring.io/).
2. Settings:
   - **Project**: Maven
   - **Language**: Java (version 21)
   - **Spring Boot**: 3.x / 4.x
   - **Group**: `com.spring.ai.ollama`
   - **Artifact**: `spring-ai-ollama`
3. Add Dependencies:
   - **Spring Web** (`spring-boot-starter-web`)
   - **Ollama** (`spring-ai-starter-model-ollama`)
4. Click **Generate**.

---

### Step 3: Add Dependency: Spring AI Ollama Starter

In your `pom.xml`, include the `spring-ai-starter-model-ollama` dependency and the `spring-ai-bom`:

```xml
<properties>
    <java.version>21</java.version>
    <spring-ai.version>2.0.1</spring-ai.version>
</properties>

<dependencies>
    <!-- Spring Web Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webmvc</artifactId>
    </dependency>

    <!-- Spring AI Ollama Starter (100% Free & Local) -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-model-ollama</artifactId>
    </dependency>

    <!-- Lombok (Optional) -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <!-- Spring AI BOM -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>${spring-ai.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

---

### Step 4: Configure Spring AI for Ollama

You can configure Ollama in either `application.properties` or `application.yml`:

#### Option A: `src/main/resources/application.properties`
```properties
spring.application.name=first-project
server.port=8081

# Ollama local daemon connection
spring.ai.ollama.base-url=http://localhost:11434
spring.ai.ollama.chat.options.model=Codellama:latest
spring.ai.ollama.chat.options.temperature=0.7
```

#### Option B: `src/main/resources/application.yml`
```yaml
spring:
  application:
    name: first-project
  ai:
    ollama:
      # Local Ollama daemon base URL
      base-url: http://localhost:11434
      chat:
        options:
          # Name of the model (e.g. Codellama:latest, llama3, mistral, deepseek-r1)
          model: Codellama:latest
          temperature: 0.7

server:
  port: 8081
```

#### Key Properties Explained:
- `base-url`: Point to your local Ollama server (`http://localhost:11434`). If running Ollama inside Docker or another machine on the LAN, replace `localhost` with the host IP.
- `model`: The exact model tag you pulled in Ollama (e.g., `Codellama:latest`, `llama3`, `mistral`, `deepseek-r1:7b`, `phi3`).
- `temperature`: Lower values (e.g. `0.2`) make responses focused and deterministic; higher values (e.g. `0.8`) make responses more creative.
- **No API Key Required**: No secrets or credentials needed!

---

### Step 5: Autowire `ChatClient.Builder` & Implement Controller

Spring AI uses the exact same `ChatClient` API abstraction for Ollama as it does for OpenAI!

In `src/main/java/com/spring/ai/firstproject/first_project/controller/ChatController.java`:

```java
package com.spring.ai.firstproject.first_project.controller;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping
public class ChatController {

    private ChatClient chatClient;

    // Injecting ChatClient.Builder (auto-configured for Ollama)
    public ChatController(ChatClient.Builder builder) {
        this.chatClient = builder.build();
    }

    @GetMapping("/chat")
    public ResponseEntity<String> chat(@RequestParam(value = "q", required = true) String query) {
        // Calls local Ollama model via Spring AI
        String response = this.chatClient.prompt(query).call().content();
        return ResponseEntity.ok(response);
    }
}
```

---

### Running and Testing Ollama

1. **Ensure Ollama is running**:
   Make sure Ollama is running in the background (`http://localhost:11434`).

2. **Start the Spring Boot Application**:
   ```bash
   .\mvnw.cmd spring-boot:run
   ```

3. **Test in Browser**:
   ```
   http://localhost:8082/ollama/chat?q=Explain microservices in simple terms
   ```

4. **Test via cURL**:
   ```bash
   curl "http://localhost:8082/ollama/chat?q=Why%20is%20the%20sky%20blue"
   ```

---

## ⚖️ OpenAI vs Ollama: Comparison

| Feature | OpenAI (Cloud) | Ollama (Local) |
| :--- | :--- | :--- |
| **Cost** | Paid (Pay per token) | **100% Free** |
| **API Key Required** | Yes (`sk-proj-...`) | **No** (Zero setup keys) |
| **Internet Required** | Yes | **No** (Works offline) |
| **Data Privacy** | Prompts sent to OpenAI cloud | **100% Private** (Data never leaves PC) |
| **Hardware Requirement** | Minimal (Cloud does compute) | Needs 8GB–16GB RAM + GPU/CPU |
| **Model Selection** | GPT-4o, GPT-4, GPT-3.5 | Llama 3, Mistral, DeepSeek, Phi-3 |
| **Spring AI Abstraction** | `ChatClient.Builder` | `ChatClient.Builder` (Same code!) |

---

## 📁 Project Structure

```
First-chatGptProject/
│
├── .mvn/wrapper/                  # Maven wrapper configuration
├── src/
│   ├── main/
│   │   ├── java/com/spring/ai/firstproject/first_project/
│   │   │   ├── FirstProjectApplication.java       # Main Spring Boot application class
│   │   │   └── controller/
│   │   │       └── ChatController.java            # ChatClient REST controller
│   │   └── resources/
│   │       └── application.yml                    # OpenAI API key and server settings
│   └── test/
│       └── java/com/spring/ai/firstproject/first_project/
│           └── FirstProjectApplicationTests.java  # Unit/Integration tests
├── .gitignore                     # Git ignore rules (ignores target/, credentials, etc.)
├── mvnw                           # Linux/macOS Maven wrapper script
├── mvnw.cmd                       # Windows Maven wrapper script
├── pom.xml                        # Maven dependencies & build configuration
└── README.md                      # Complete guide for OpenAI & Ollama
```

---

## 📤 Deploying / Pushing to GitHub

To commit and push updates to GitHub:

```bash
# Check status
git status

# Stage updated documentation and files
git add .

# Commit changes
git commit -m "docs: add Step 2 - Spring AI with Ollama (Free Local LLM) guide"

# Push to GitHub
git push origin main
```
