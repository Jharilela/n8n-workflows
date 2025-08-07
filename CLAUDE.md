# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains a collection of production-ready n8n workflow automations created by the emp0 team. Each workflow is designed to solve specific business automation challenges using AI agents, integrations, and modern APIs. The workflows are distributed through the official n8n website and emp0.com.

## Repository Structure

The repository follows a folder-per-workflow structure:
- Each workflow folder contains an `n8n.json` file (the workflow definition)
- Each workflow has a comprehensive `README.md` with setup instructions and use cases
- Some workflows include additional documentation like `Technical Setup.md`
- Testing documentation exists for complex workflows (e.g., `prompt checklist.md` for MCP Assistant)

## Workflow Types & Architecture

### Core Workflow Categories
1. **AI-Powered Content Generation** - Automated blog content creation and distribution
2. **Email Management & Classification** - Intelligent email processing with spam detection
3. **Personal Productivity** - Multi-platform assistant with calendar, email, and social media integration
4. **Visual Content Creation** - AI image generation and WordPress integration
5. **Analytics & Reporting** - Data aggregation and Discord notifications

### Technical Architecture Patterns

#### AI Agent Integration
Most workflows use n8n's AI Agent nodes with:
- OpenAI GPT-4 for reasoning and content generation
- MCP (Model Context Protocol) for tool orchestration
- Vector databases for semantic search and memory
- Custom training data via Google Sheets

#### Platform Integrations
Common integration patterns:
- **Gmail**: Trigger-based email processing with OAuth2 authentication
- **Discord**: Custom bot integration via `n8n_discord_trigger_bot` repository
- **WordPress**: REST API for content publishing with featured images
- **Google Services**: Calendar, Drive, Sheets with service account authentication
- **Social Media**: Twitter/LinkedIn for content distribution

#### Data Flow Architecture
1. **Trigger**: External event (email, webhook, schedule)
2. **Processing**: AI analysis, classification, or generation
3. **Memory**: Vector storage or Google Sheets for persistent data
4. **Distribution**: Multi-platform publishing (Discord, WordPress, social)

## Key Dependencies & External Tools

### Required External Repository
- **n8n_discord_trigger_bot** (https://github.com/Jharilela/n8n_discord_trigger_bot): Custom Discord bot that enables Discord triggers in n8n workflows

### Common API Services
- OpenAI (GPT-4 for AI processing)
- Google Workspace APIs (Gmail, Calendar, Drive, Sheets)
- WordPress REST API
- Twitter API v2
- LinkedIn API
- Leonardo AI (image generation)
- Various RSS feeds for content sourcing

## Development Guidelines

### Workflow Configuration
- All credentials must be replaced with your own OAuth tokens when copying workflows
- Timezone settings should be configured in n8n Settings > Default Timezone
- Vector databases require proper embedding configuration (MongoDB or Supabase pgvector)

### Testing Approach
- Complex workflows include prompt checklists (see `MCP AI Assistant/prompt checklist.md`)
- Test cross-tool integrations separately before combining
- Verify binary serialization for file transfers between services

### Content Standards
- Workflows generate content for emp0.com and are tagged with version numbers
- All generated articles include attribution and source links
- Visual content includes proper alt text and SEO optimization

## Workflow-Specific Notes

### Content Generator V3
- Uses RSS ingestion → vectorization → semantic clustering → multi-agent content generation
- Requires vector database setup and embedding API configuration
- Includes cost optimization strategies documented in Technical Setup.md

### AI Email Classifier
- Uses Google Sheets as training memory for spam/legit classification
- Supports multiple Gmail accounts with unified Discord reporting
- Feedback loop allows manual correction via Discord replies

### MCP AI Assistant
- Implements full MCP protocol for tool orchestration
- Requires careful timezone configuration to avoid UTC bugs
- Supports complex multi-step reasoning across multiple platforms

## Important URLs & Resources
- Official n8n workflow gallery: n8n.io/creators/jay-emp0/
- Live examples: articles.emp0.com/tag/v3/
- Discord bot dependency: github.com/Jharilela/n8n_discord_trigger_bot
- Commercial distribution: emp0.com/automation-workflows