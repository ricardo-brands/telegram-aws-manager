# Super DevOps Bot: AWS & Server Management via Telegram with n8n + Gemini AI

An advanced flow built in **n8n** that turns Telegram into a complete command center for DevOps operations. 

This project combines the strict, deterministic control of cloud infrastructure (AWS EC2) with the flexibility of Artificial Intelligence (Google Gemini) for internal Linux operating system management via SSH, using natural language.

---

## 🎯 What does this bot do?

The flow operates on two main fronts:

1. **AWS Infrastructure Management (Hard-coded & Secure):**
   Exact commands that call the AWS API to manage EC2 instances. There is no room for AI hallucinations here.
   - Start, Stop, and Reboot instances.
   - Check Status (IP, Current state, Hardware type).
   - Change the machine size (e.g., `t2.micro` to `t3.medium`) automatically when the machine is stopped.

2. **OS Management via SSH (Powered by AI):**
   Secure access to the Linux terminal where the user asks for what they want in *natural language*. The AI translates it into a secure command, n8n executes it via SSH, and the AI "chews" the technical log, returning a friendly response on Telegram.
   - Ex: *"How is the memory consumption?"* -> Executes `free -m`.
   - Ex: *"Which sites are configured in Apache?"* -> Lists the `sites-enabled` directory.
   - Ex: *"Restart the database container"* -> Executes `docker-compose restart`.

---

## ⚙️ Architecture and Node Logic (n8n)

The flow was designed with the **Master Routing** concept, separating rigid commands from AI requests.

### 🛡️ 1. Entry & Security Layer
* **Telegram Trigger:** Listens to all messages sent to the bot.
* **Verify Users (If):** A vital security filter. The flow only advances if the sender's `chat.id` is in the authorized IDs list. Requests from strangers are ignored.

### 🧠 2. The Routing Brain
* **Logic Brain (Code Node):** The maestro of the orchestra. It analyzes the first term of the message. If it's an infrastructure command (e.g., `/ligar`, `/mudar`), it routes to AWS. If it's `/ajuda`, it displays the menu. If it's any text in natural language, it classifies it as `ssh` and sends it to the AI.
* **Master Router (Switch):** Physically distributes the flow to the 6 possible paths based on the Logic Brain's decision.

### ☁️ 3. Infrastructure Arm (AWS EC2 API)
* **HTTP Request Nodes (AWS Start, Stop, etc.):** Authenticated via AWS IAM, they send secure requests via `POST` (`form-urlencoded`) to the EC2 API.
* **Response: Status / Executed Action (Telegram):** Extracts data from the XML returned by AWS (using Regex) and returns the actual status or success confirmation to the user.

### 🤖 4. Operating System Arm (SSH + Gemini AI)
* **AI - Understands Request (Chain LLM):** Receives free text from the user and evaluates it under a *Maximum Security* prompt. It tries to map the intent to a strict list of predefined Linux commands. If the intent is destructive or out of scope, it returns "ERROR".
* **Command Allowed? (If):** Checks if the AI generated a valid command or blocked the execution.
* **SSH - Execute:** Accesses the server via private key (secure authentication) and executes the terminal command.
* **AI - Explains Result:** Takes the `stdout` (technical Linux log) and uses Gemini again to translate the technical jargon into a clear summary, formatted for Telegram (without using list asterisks so it doesn't break the messenger's API).
* **Send AI Response (Telegram):** Delivers the final diagnosis to the user.

---

## 🛠️ Prerequisites

To import and run this flow in your n8n instance, you will need:

1. **Telegram Bot Token:** Created via *BotFather*.
2. **AWS IAM Credentials:** An AWS user with restricted policies (Least Privilege) for `ec2:DescribeInstances`, `ec2:StartInstances`, `ec2:StopInstances`, `ec2:RebootInstances`, and `ec2:ModifyInstanceAttribute`.
3. **SSH Private Key:** For access to the target server.
4. **Google Gemini API Key:** For natural language processing (Google AI Studio's free tier is sufficient).
5. (Optional) **n8n v1.0+**: The flow uses the updated `Switch` and `LangChain` nodes.

---

## 💡 How to Setup

1. Import the `workflow.json` file into your n8n.
2. Add your credentials in all corresponding nodes (Telegram, AWS, SSH, Google Gemini).
3. In the **Verify Users** node, replace the value `YOUR_CHAT_ID_HERE` with your Telegram Chat ID (and add your team's IDs).
4. In the **Logic Brain** node, change the `instance_id` variable to your EC2 machine ID and set your `regiao` (e.g., `sa-east-1`).
5. In the **AI - Understands Request** node, edit the Prompt to include the actual paths of your Docker containers, services, or scripts that the AI is allowed to run.

---

## 💬 Usage Examples on Telegram

**For Infrastructure:**
> `/status` -> Returns State, IP, and Hardware Type.  
> `/desligar` -> Stops the machine for maintenance.  
> `/mudar t3.medium` -> Upgrades the server size.

**For the Operating System:**
> "How is the disk space?" -> *The AI logs in, runs `df -h` and replies nicely.* > "Which domains are running on Apache?" -> *The AI runs the vhost check and lists the sites.* > "Restart the ERP container for me" -> *The AI finds the correct path, runs docker-compose restart, and notifies you when it's back.*

---

## 🤝 Contributing
Feel free to fork the project, create pull requests with new commands, or improve the AI protection prompts.

Developed to simplify the lives of SysAdmins and DevOps Engineers. Built for the trenches! 💀
