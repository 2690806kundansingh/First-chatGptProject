# Spring AI Setup with OpenAI

A step-by-step guide and working implementation of **Spring Boot** integrated with **Spring AI** and **OpenAI (ChatGPT)**.

---

## 📌 Table of Contents
1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Step 1: Generate an OpenAI API Key](#step-1-generate-an-openai-api-key)
4. [Step 2: Create a Spring Boot Project](#step-2-create-a-spring-boot-project)
5. [Step 3: Add Dependencies in `pom.xml`](#step-3-add-dependencies-in-pomxml)
6. [Step 4: Configure API Key in `application.yml`](#step-4-configure-api-key-in-applicationyml)
7. [Step 5: Autowire `ChatClient.Builder` & Create ChatController](#step-5-autowire-chatclientbuilder--create-chatcontroller)
8. [Running and Testing the Application](#running-and-testing-the-application)
9. [Project Structure](#project-structure)
10. [Deploying / Pushing to GitHub](#deploying--pushing-to-github)

---

## 📖 Overview
This project demonstrates how to connect Spring Boot with OpenAI's large language models (LLM) using the official **Spring AI** framework. It provides a simple REST API endpoint (`/chat?q={your_question}`) that accepts a user prompt, forwards it to OpenAI, and returns the AI's response.

---

## 🛠 Prerequisites
- **Java 21** or later
- **Maven 3.8+** (or use the included `./mvnw` wrapper)
- An active **OpenAI Account** with API credits
- Your preferred IDE (IntelliJ IDEA, VS Code, Eclipse)

---

## 🔑 Step 1: Generate an OpenAI API Key

To communicate with OpenAI's models, you need an API secret key:

1. Go to the [OpenAI Platform](https://platform.openai.com/).
2. Log in or create an account.
3. In the left navigation, click on **API Keys** (or visit [https://platform.openai.com/api-keys](https://platform.openai.com/api-keys)).
4. Click **+ Create new secret key**.
5. Give your key an identifiable name (e.g., `SpringAiKey`) and click **Create secret key**.
6. **Copy the key immediately** and store it in a secure location (OpenAI will not show it to you again).

### 🔒 Best Practice: Set as Environment Variable
Never hardcode your API key into source code or commit it to GitHub. Instead, store it as an environment variable named `OPENAI_API_KEY`:

- **Windows (PowerShell)**:
  ```powershell
  # Temporary (current session)
  $env:OPENAI_API_KEY="sk-proj-xxxxxxxxxxxxxxxxxxxxxxxx"

  # Permanent (user level)
  [System.Environment]::SetEnvironmentVariable('OPENAI_API_KEY', 'sk-proj-xxxxxxxxxxxxxxxxxxxxxxxx', 'User')
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

## 🚀 Step 2: Create a Spring Boot Project

You can bootstrap your project using [Spring Initializr](https://start.spring.io/):

1. Open [start.spring.io](https://start.spring.io/).
2. Choose the following settings:
   - **Project**: Maven
   - **Language**: Java
   - **Spring Boot**: 3.x or 4.x
   - **Group**: `com.spring.ai.firstproject`
   - **Artifact**: `first-project`
   - **Name**: `first-project`
   - **Packaging**: Jar
   - **Java Version**: 21
3. In **Dependencies**, add:
   - **Spring Web** (`spring-boot-starter-web` or `spring-boot-starter-webmvc`)
   - **OpenAI** (`spring-ai-starter-model-openai`)
   - **Lombok** (optional)
4. Click **Generate** to download the zip file and extract it.

---

## 📦 Step 3: Add Dependencies in `pom.xml`

Ensure your `pom.xml` contains the required Spring AI dependencies and dependency management BOM:

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

## ⚙️ Step 4: Configure API Key in `application.yml`

Create or update `src/main/resources/application.yml` with the following configuration:

```yaml
spring:
  application:
    name: first-project
  ai:
    openai:
      # Reads the key securely from your system environment variable
      api-key: ${OPENAI_API_KEY}

server:
  # Port on which the Spring Boot server runs
  port: 8081
```

> **Why use `${OPENAI_API_KEY}`?**
> Referencing the environment variable `${OPENAI_API_KEY}` keeps your private secret key safe and prevents accidental exposure when pushing to public repositories like GitHub.

---

## 💻 Step 5: Autowire `ChatClient.Builder` & Create ChatController

Spring AI provides a high-level, fluent API called `ChatClient` to interact with LLMs. Spring AI automatically provides an auto-configured `ChatClient.Builder` bean.

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
        // Send user query to OpenAI and retrieve the response content
        var resultResponse = chatClient.prompt(q).call().content();
        return ResponseEntity.ok(resultResponse);
    }
}
```

### Explanation:
1. **`@Autowired ChatClient.Builder builder`**: Spring Boot injects the preconfigured `ChatClient.Builder` configured with the OpenAI connection details from `application.yml`.
2. **`builder.build()`**: Instantiates the immutable `ChatClient` instance.
3. **`chatClient.prompt(q).call().content()`**:
   - `prompt(q)`: Passes the question string to the chat client.
   - `call()`: Executes the HTTP API call to OpenAI.
   - `content()`: Extracts the assistant's text answer.

---

## 🏃 Running and Testing the Application

### 1. Start the Application
Run the following command from the project root:

```bash
# Using Maven wrapper (Windows PowerShell / CMD)
.\mvnw.cmd spring-boot:run

# Using Maven wrapper (macOS / Linux)
./mvnw spring-boot:run
```

The application will start on port `8081`.

### 2. Test via Web Browser
Open your browser and navigate to:
```
http://localhost:8081/chat?q=Tell me a short joke about programming
```

### 3. Test via cURL
```bash
curl "http://localhost:8081/chat?q=Explain%20Spring%20Boot%20in%20one%20sentence"
```

### 4. Sample Response
```text
Spring Boot is an open-source, convention-over-configuration Java framework designed to simplify the setup, development, and deployment of production-ready, standalone Spring applications.
```

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
└── README.md                      # Project documentation and guide
```

---

## 📤 Deploying / Pushing to GitHub

To push this project to GitHub:

```bash
# 1. Initialize git (if not already initialized)
git init -b main

# 2. Add remote repository
git remote add origin https://github.com/2690806kundansingh/First-chatGptProject.git

# 3. Stage all files
git add .

# 4. Commit changes
git commit -m "Initial commit: Spring AI with OpenAI ChatClient setup and documentation"

# 5. Push to GitHub
git push -u origin main
```
