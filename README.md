# uAgent-A2A Adapter

## Introduction
The uAgent-A2A Adapter is a revolutionary bridge between [Fetch.ai](https://fetch.ai)'s uAgent framework and the A2A (Agent-to-Agent) Protocol. This adapter creates a seamless communication layer that enables decentralized AI agents to collaborate across different platforms and protocols, leveraging Fetch.ai’s autonomous agent infrastructure while extending compatibility to the broader A2A ecosystem.

## What is the uAgent-A2A Adapter?
The uAgent-A2A Adapter is a comprehensive Python module designed to integrate A2A systems with Fetch.ai’s uAgents, enabling intelligent multi-agent coordination and communication.

### Overview
The adapter provides a seamless bridge between A2A agents and the uAgent ecosystem, offering:
- **Multi-Agent Coordination**: Manage multiple specialized AI agents from a single interface.
- **Intelligent Routing**: Automatically route queries to the most suitable agent based on keywords, specialties, or LLM-based analysis.
- **Health Monitoring**: Continuous health checking and discovery of available agents.
- **Fallback Mechanisms**: Robust error handling with fallback executors.
- **Chat Protocol Integration**: Full support for uAgent chat protocols and messaging.

### Features
- **Multi-Agent Management**:
  - Configure and manage multiple A2A agents with different specialties.
  - Automatic agent discovery and registration.
  - Health monitoring and status tracking.
- **Intelligent Routing**:
  - **Keyword Matching**: Route queries based on agent keywords and specialties.
  - **LLM-Based Routing**: Use AI to intelligently select the best agent for complex queries.
  - **Round-Robin**: Distribute load evenly across available agents.
  - **Priority-Based**: Assign priorities to agents for preferential routing.
- **Communication Protocols**:
  - Full uAgent chat protocol support.
  - Asynchronous message handling.
  - Acknowledgment and error handling.
  - Real-time agent communication.
- **Reliability Features**:
  - Health checking and agent discovery.
  - Fallback executor support.
  - Graceful error handling.
  - Timeout management.

## Architecture
The uAgent-A2A Adapter serves as a translation layer between:
- **uAgent Protocol**: Fetch.ai’s native agent communication system.
- **A2A Protocol**: Universal agent-to-agent communication standard.

```
uAgent (Fetch.ai) ←→ uAgent-A2A Adapter ←→ A2A Agent (Any Platform)
```

### Core Components
1. **A2AAgentConfig** – Agent Configuration Management
   ```python
   from dataclasses import dataclass
   from typing import List, Optional

   @dataclass
   class A2AAgentConfig:
       name: str
       description: str
       url: str
       port: int
       specialties: List[str]
       skills: Optional[List[str]] = None
       examples: Optional[List[str]] = None
       keywords: Optional[List[str]] = None
       priority: int = 1
   ```
   - **Purpose**: Defines how each agent in the network is characterized and discovered.
   - **Key Features**:
     - Auto-generation of skills, examples, and keywords from specialties.
     - Dynamic keyword extraction for searchable attributes.
     - Priority system for routing decisions.

2. **uAgent Integration Layer**
   - Acts as a bridge between application logic and the uAgent framework.
   - Enables decentralized, asynchronous communication between agents.
   ```python
   from uagents import Agent, Context, Protocol
   from uagents_core.contrib.protocols.chat import (
       ChatMessage, ChatAcknowledgement, TextContent, chat_protocol_spec
   )

   self.uagent = Agent(
       name=name,
       port=port,
       seed=self.seed,
       mailbox=mailbox
   )
   self.chat_proto = Protocol(spec=chat_protocol_spec)
   ```

3. **A2A Communication Layer**
   - Facilitates structured messaging between autonomous agents using HTTP protocols.
   ```python
   import httpx
   from uuid import uuid4

   payload = {
       "id": uuid4().hex,
       "params": {
           "message": {
               "role": "user",
               "parts": [{"type": "text", "text": user_message}],
               "messageId": uuid4().hex,
           },
       }
   }
   response = await httpx_client.post(
       f"http://localhost:{self.a2a_port}/send-message",
       json=payload,
       headers={"Content-Type": "application/json"}
   )
   ```

4. **Intelligent Message Routing**
   - **Keyword-Based Routing**:
     ```python
     async def _route_by_keywords(self, query: str, agents: List[Dict], ctx: Context):
         # Scores agents based on keyword matches
         # Higher scores for specialty matches
         # Word overlap analysis for better matching
     ```
   - **LLM-Powered Routing**:
     ```python
     async def _llm_route_query(self, query: str, agents: List[Dict], ctx: Context):
         # Uses AI to intelligently select the best agent
         # Analyzes query context and agent capabilities
         # Provides fallback to keyword matching
     ```

### Message Flow
1. **Incoming Message**: External uAgent sends a chat message.
2. **Agent Discovery**: Adapter discovers and health-checks available agents.
3. **Query Routing**: Router selects the best agent based on strategy.
4. **Message Forwarding**: Query sent to the selected A2A agent.
5. **Response Processing**: Agent response processed and formatted.
6. **Reply**: Response sent back to the original sender.
7. **Acknowledgment**: Confirmation sent to complete the cycle.

## Key Functions
1. **`handle_message()`** – The Message Processing Hub
   ```python
   @self.chat_proto.on_message(ChatMessage)
   async def handle_message(ctx: Context, sender: str, msg: ChatMessage):
       # Receives messages, routes to specialist agent, handles responses
   ```
   - Central hub for all incoming messages, managing communication flow.

2. **`_route_query()`** – The Intelligence Director
   ```python
   async def _route_query(self, query: str, ctx: Context) -> Optional[Dict[str, Any]]:
       # Analyzes query content and matches to agent capabilities
   ```
   - Ensures queries are routed to the most qualified agent.

3. **`_send_to_a2a_agent()`** – The Communication Bridge
   ```python
   async def _send_to_a2a_agent(self, message: str, a2a_url: str) -> str:
       # Translates uAgent messages to A2A format and sends them
   ```
   - Handles translation between different agent communication protocols.

4. **`_discover_and_health_check_agents()`** – The Network Monitor
   ```python
   async def _discover_and_health_check_agents(self, ctx: Context = None):
       # Discovers and monitors the health of A2A agents
   ```
   - Ensures only healthy, available agents are used for routing.

## Health Monitoring
- Automatically monitors agent health and excludes unhealthy agents from routing.
- Checks include:
  - Agent card availability at `/.well-known/agent.json`.
  - HTTP response status.
  - Response time monitoring.

## Error Handling
- **Agent Unavailable**: Routes to alternative agents.
- **Network Timeouts**: Configurable timeout settings with graceful degradation.
- **Invalid Responses**: Falls back to error messages or alternative agents.
- **Health Check Failures**: Automatic agent exclusion and retry logic.

## Step-by-Step Process
### Phase 1: Initialization and Discovery
1. **uAgent Startup**:
   ```python
   @self.uagent.on_event("startup")
   async def on_start(ctx: Context):
       ctx.logger.info(f"🚀 A2A uAgent started at address: {self.uagent.address}")
       await self._discover_and_health_check_agents(ctx)
       self.uagent.include(self.chat_proto, publish_manifest=True)
   ```

2. **A2A Agent Discovery**:
   ```python
   async def _discover_and_health_check_agents(self, ctx: Context = None):
       async with httpx.AsyncClient(timeout=10.0) as client:
           for config in self.agent_configs:
               card_url = f"{config.url}/.well-known/agent.json"
               response = await client.get(card_url)
               if response.status_code == 200:
                   agent_card = response.json()
                   self.discovered_agents[config.name] = {
                       "name": config.name,
                       "url": config.url,
                       "specialties": config.specialties,
                       "card": agent_card
                   }
   ```

### Phase 2: Message Reception and Processing
3. **uAgent Receives Message**:
   ```python
   @self.chat_proto.on_message(ChatMessage)
   async def handle_message(ctx: Context, sender: str, msg: ChatMessage):
       for item in msg.content:
           if isinstance(item, TextContent):
               ctx.logger.info(f"📩 Received from {sender}: {item.text}")
               best_agent = await self._route_query(item.text, ctx)
               response = await self._send_to_a2a_agent(item.text, best_agent.get("url"))
   ```

4. **Intelligent Routing Decision**:
   ```python
   async def _route_query(self, query: str, ctx: Context) -> Optional[Dict[str, Any]]:
       llm_selected_agent = await self._llm_route_query(query, agents, ctx)
       if llm_selected_agent:
           return llm_selected_agent
       return await self._route_by_keywords(query, healthy_agents, ctx)
   ```

### Phase 3: A2A Communication
5. **Message Translation and Sending**:
   ```python
   async def _send_to_a2a_agent(self, message: str, a2a_url: str) -> str:
       payload = {
           "id": uuid4().hex,
           "params": {
               "message": {
                   "role": "user",
                   "parts": [{"type": "text", "text": message}],
                   "messageId": uuid4().hex,
               },
           }
       }
       for endpoint in ["/send-message", "/", "/message", "/chat"]:
           response = await httpx_client.post(
               f"{a2a_url}{endpoint}",
               json=payload,
               headers={"Content-Type": "application/json"}
           )
           if response.status_code == 200:
               return self._parse_a2a_response(response.json())
   ```

6. **A2A Response Processing**:
   ```python
   def _parse_a2a_response(self, result: dict) -> str:
       if "result" in result:
           result_data = result["result"]
           if "artifacts" in result_data:
               full_text = ""
               for artifact in result_data["artifacts"]:
                   if "parts" in artifact:
                       for part in artifact["parts"]:
                           if part.get("kind") == "text":
                               full_text += part.get("text", "")
               return full_text.strip()
           elif "parts" in result_data and len(result_data["parts"]) > 0:
               return result_data["parts"][0].get("text", "").strip()
       return "✅ Response received from A2A agent"
   ```

### Phase 4: Response Delivery
7. **Send Response Back to uAgent**:
   ```python
   response_msg = ChatMessage(
       timestamp=datetime.now(timezone.utc),
       msg_id=uuid4(),
       content=[TextContent(type="text", text=response)]
   )
   await ctx.send(sender, response_msg)
   ack_msg = ChatAcknowledgement(
       timestamp=datetime.now(timezone.utc),
       acknowledged_msg_id=msg.msg_id
   )
   await ctx.send(sender, ack_msg)
   ```

## Advanced Features
1. **Multi-Agent Orchestration**:
   ```python
   agent_configs = [
       A2AAgentConfig(
           name="DataAnalyst",
           description="Specialized in data analysis and statistics",
           url="http://localhost:9001",
           port=9001,
           specialties=["data analysis", "statistics", "visualization"],
           priority=2
       ),
       A2AAgentConfig(
           name="ContentWriter",
           description="Expert in content creation and writing",
           url="http://localhost:9002",
           port=9002,
           specialties=["writing", "content creation", "copywriting"],
           priority=1
       )
   ]
   adapter = A2AAdapter(
       name="MultiAgentSystem",
       agent_configs=agent_configs,
       routing_strategy="keyword_match"
   )
   ```

2. **Health Monitoring and Failover**:
   ```python
   async def _discover_and_health_check_agents(self):
       for config in self.agent_configs:
           card_url = f"{config.url}/.well-known/agent.json"
           response = await client.get(card_url)
           if response.status_code == 200:
               self.agent_health[config.name] = True
               print(f"✅ Agent {config.name} is healthy")
           else:
               self.agent_health[config.name] = False
               print(f"❌ Agent {config.name} health check failed")
   ```

3. **LLM-Powered Intelligent Routing**:
   ```python
   async def _llm_route_query(self, query: str, agents: List[Dict]) -> Optional[Dict]:
       agent_descriptions = []
       for i, agent in enumerate(agents):
           agent_desc = f"{i+1}. {agent.get('name')}: {agent.get('description')} - Specializes in: {', '.join(agent.get('specialties', []))}"
           agent_descriptions.append(agent_desc)
       prompt = f"""
       Available Agents:
       {chr(10).join(agent_descriptions)}
       User Query: "{query}"
       Select the best agent (return only the number):
       """
       response = requests.post(
           "https://api.asi1.ai/v1/chat/completions",
           headers={'Authorization': 'Bearer YOUR_API_KEY'},
           json={
               "model": "asi1-mini",
               "messages": [{"role": "user", "content": prompt}],
               "temperature": 0,
               "max_tokens": 10
           }
       )
       if response.status_code == 200:
           llm_response = response.json()['choices'][0]['message']['content'].strip()
           agent_index = int(llm_response) - 1
           return agents[agent_index] if 0 <= agent_index < len(agents) else None
   ```

## Conclusion
The uAgent-A2A Adapter is a significant advancement in decentralized AI communication, combining Fetch.ai’s uAgent framework with the A2A protocol. This integration enables the creation of sophisticated, economically-aware agent networks that operate autonomously while maintaining interoperability with diverse AI systems. Developers can leverage this adapter to build the next generation of decentralized AI applications that are both intelligent and economically sustainable, paving the way for a truly autonomous digital economy.
