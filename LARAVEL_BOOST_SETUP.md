# Laravel Boost Setup Documentation

## Overview

Laravel Boost has been successfully installed and configured for AI-assisted Laravel development.

**Installation Date:** October 24, 2025  
**Laravel Boost Version:** 1.5.0  
**Laravel Version:** 12.35.0  
**PHP Version:** 8.4.5

---

## What is Laravel Boost?

Laravel Boost is an official Laravel package that provides an MCP (Model Context Protocol) server for AI-assisted development. It bridges the gap between AI coding assistants and your Laravel application by providing:

- **Real-time app context** - AI agents can query your database, routes, config, logs
- **17,000+ docs** - Vectorized Laravel ecosystem documentation with semantic search
- **AI guidelines** - Curated coding guidelines from Laravel maintainers
- **15+ MCP tools** - Specialized tools for Laravel development

---

## Installed Components

### 1. Composer Packages

```json
{
  "require-dev": {
    "laravel/boost": "^1.5",
    "laravel/mcp": "^0.3.0",
    "laravel/roster": "^0.2.9"
  }
}
```

**laravel/boost** - Main package providing MCP server and commands  
**laravel/mcp** - MCP protocol implementation  
**laravel/roster** - Package management and version tracking

### 2. Configuration Files

#### `.mcp.json` (Claude Code Configuration)
```json
{
  "mcpServers": {
    "laravel-boost": {
      "command": "php",
      "args": ["artisan", "boost:mcp"]
    }
  }
}
```

#### `.cursor/mcp.json` (Cursor IDE Configuration)
```json
{
  "mcpServers": {
    "laravel-boost": {
      "command": "php",
      "args": ["artisan", "boost:mcp"]
    }
  }
}
```

#### `boost.json` (Boost Configuration)
```json
{
  "agents": ["claude_code", "cursor"],
  "editors": ["claude_code", "cursor"],
  "guidelines": []
}
```

#### `CLAUDE.md` (AI Guidelines)
Comprehensive AI guidelines including:
- Laravel 12 conventions
- Package versions and compatibility
- Code structure rules
- Testing guidelines
- Frontend bundling instructions

#### `.cursor/rules/laravel-boost.mdc`
Cursor-specific AI guidelines (12KB)

---

## MCP Tools Available

Laravel Boost provides 15+ specialized tools:

### Application Information
- **application-info** - Get app details, environment, versions

### Documentation
- **search-docs** - Semantic search through Laravel docs
- **list-docs** - List available documentation packages

### Database
- **database-query** - Execute database queries
- **database-schema** - View database schema and tables

### Development
- **tinker** - Execute PHP code in Tinker REPL
- **list-artisan-commands** - List all Artisan commands
- **list-routes** - View application routes

### Configuration
- **read-config** - Read configuration values
- **read-env** - Read environment variables

### Debugging
- **browser-logs** - View browser console logs
- **last-errors** - Get recent error logs
- **read-logs** - Read application logs

### Feedback
- **report-feedback** - Send feedback to Laravel Boost team

---

## AI Guidelines Loaded

The following guidelines were installed:

1. **boost** - Core Boost guidelines
2. **foundation** - Laravel foundation rules
3. **laravel/core** - Laravel framework guidelines
4. **laravel/v12** - Laravel 12 specific rules
5. **pest/core** - Pest testing framework
6. **php** - PHP best practices
7. **pint/core** - Laravel Pint code style

### Package Versions Detected

```
- php: 8.4.5
- laravel/framework (LARAVEL): v12
- laravel/prompts (PROMPTS): v0
- laravel/sanctum (SANCTUM): v4
- laravel/mcp (MCP): v0
- laravel/pint (PINT): v1
- laravel/sail (SAIL): v1
- pestphp/pest (PEST): v3
- phpunit/phpunit (PHPUNIT): v11
```

---

## Using Laravel Boost

### In Claude Code

Laravel Boost is automatically configured via `.mcp.json`. The MCP server starts when you open the project.

**Available commands:**
- Ask questions about your app structure
- Request database queries
- Search Laravel documentation
- Execute Tinker commands
- View routes and configs

**Example prompts:**
- "Show me all routes in the application"
- "What's in the database schema?"
- "Search Laravel docs for validation rules"
- "Execute this query in Tinker: User::count()"

### In Cursor

Cursor AI can access Laravel Boost through `.cursor/mcp.json` configuration.

**Usage:**
1. Open Cursor IDE
2. Access MCP settings
3. Enable `laravel-boost` server
4. Use AI assistant with Laravel context

### Manual Testing

```bash
# Start MCP server manually
php artisan boost:mcp

# Update guidelines to latest
php artisan boost:update

# Reinstall Boost
php artisan boost:install
```

---

## Artisan Commands

Laravel Boost provides these Artisan commands:

```bash
# Install/configure Boost (already done)
php artisan boost:install

# Start MCP server (usually called automatically by IDE)
php artisan boost:mcp

# Update AI guidelines to latest versions
php artisan boost:update
```

---

## Benefits for Development

### 1. Accurate Code Generation
- AI understands your exact Laravel version (12.35.0)
- Knows installed packages and their versions
- Follows Laravel conventions automatically

### 2. Context-Aware Suggestions
- Can query your database structure
- Knows your routes and middleware
- Understands your application config

### 3. Reduced Errors
- Fewer hallucinations
- Framework-appropriate code
- Version-compatible syntax

### 4. Faster Development
- No need to explain Laravel basics
- Instant access to documentation
- Real-time app state inspection

---

## Keeping Boost Updated

### Automatic Updates (Recommended)

Add to `composer.json`:

```json
{
  "scripts": {
    "post-update-cmd": [
      "@php artisan vendor:publish --tag=laravel-assets --ansi --force",
      "@php artisan boost:update --ansi"
    ]
  }
}
```

This ensures guidelines are updated after every `composer update`.

### Manual Updates

```bash
# Update Boost package
composer update laravel/boost

# Update guidelines
php artisan boost:update
```

---

## Verification

### Check Installation

```bash
# Verify Boost is installed
composer show laravel/boost

# List Boost commands
php artisan list boost

# Verify app is working
php artisan about
```

### Test MCP Server

```bash
# Start MCP server in test mode
php artisan boost:mcp
```

The server should start and listen for MCP protocol messages.

---

## File Structure

```
project-root/
├── .mcp.json                    # Claude Code MCP config
├── .cursor/
│   ├── mcp.json                 # Cursor MCP config
│   └── rules/
│       └── laravel-boost.mdc    # Cursor AI guidelines
├── boost.json                   # Boost configuration
├── CLAUDE.md                    # AI guidelines document
├── composer.json                # Boost dependencies
└── vendor/
    └── laravel/
        ├── boost/               # Boost package
        ├── mcp/                 # MCP protocol
        └── roster/              # Package roster
```

---

## Troubleshooting

### MCP Server Not Starting

```bash
# Check if artisan is working
php artisan --version

# Verify Boost is installed
composer show laravel/boost

# Reinstall Boost
php artisan boost:install
```

### Guidelines Not Loading

```bash
# Update guidelines manually
php artisan boost:update

# Check boost.json configuration
cat boost.json
```

### IDE Not Detecting MCP Server

**Claude Code:**
- Check `.mcp.json` exists in project root
- Restart Claude Code

**Cursor:**
- Check `.cursor/mcp.json` exists
- Open MCP settings and enable `laravel-boost`
- Restart Cursor

---

## Resources

- [Laravel Boost Documentation](https://boost.laravel.com)
- [Laravel Boost GitHub](https://github.com/laravel/boost)
- [MCP Protocol Specification](https://modelcontextprotocol.io)
- [Laravel 12 Documentation](https://laravel.com/docs/12.x)

---

## Git Commits

Laravel Boost installation was committed in 2 organized commits:

**Commit 1: Dependencies**
```
85e9339 - feat: add Laravel Boost for AI-assisted development
- composer.json
- composer.lock
```

**Commit 2: Configuration**
```
114b0a6 - feat: configure Laravel Boost MCP server and AI guidelines
- .mcp.json
- .cursor/mcp.json
- .cursor/rules/laravel-boost.mdc
- boost.json
- CLAUDE.md
```

---

## Next Steps

1. ✅ **Installation Complete** - Laravel Boost is ready to use
2. ✅ **IDE Configured** - Claude Code and Cursor support enabled
3. ✅ **Guidelines Loaded** - 7 AI guidelines active

**Optional:**
- Explore MCP tools with your AI assistant
- Add automatic updates to composer.json
- Share guidelines with your team
- Provide feedback via `report-feedback` tool

---

**Status:** ✅ Laravel Boost is fully installed and configured!

Your AI coding assistants now have deep Laravel context and can provide more accurate, framework-appropriate suggestions.
