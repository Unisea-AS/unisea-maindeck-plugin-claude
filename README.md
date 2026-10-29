# UniSea Maindeck

UniSea Maindeck is a project management system for ship owners and shipyards. It plans and runs dry-docking and ship repair projects: write the specification, tender it to shipyards, run the yard visit and close against the yard's invoice. This plugin connects Claude to your Maindeck account so you can work with your team's projects from Claude.

## What you can do

- List and open projects, work orders, inspections and owner supply items.
- Get project statistics and tender status.
- Create projects and work orders, and update their fields.
- Add and update cost items on work orders.
- Add actions (to-dos) and mark them done.
- Add users to a project and change their project role.

Claude sees only the data that your Maindeck role allows. A Maindeck account is required.

## What the plugin contains

- **MCP server:** connects to the Maindeck MCP server at `https://app.maindeck.io/api/mcp`. You sign in with your Maindeck account (OAuth). All requests go to that server only, and run as the signed-in user.
- **Skill `maindeck-mcp`:** teaches Claude the Maindeck terms, phases, statuses and writing rules, and tells it to confirm every write before it is made. The MCP server has no delete tools.

The plugin runs no local code and no hooks, and sends no data anywhere other than the Maindeck MCP server.

## Support

- Website: https://www.maindeck.io
- Support: https://support.unisea.no
- Privacy policy: https://www.unisea.no/privacy-policy
- Terms: https://www.unisea.no/terms-and-conditions

## License

The plugin files in this repository are licensed under the Apache License 2.0 (see `LICENSE` and `NOTICE`). "UniSea" and "Maindeck" are trademarks of Unisea AS and are not licensed. Use of the UniSea Maindeck service is governed by the [UniSea Terms of Use](https://www.unisea.no/terms-and-conditions).
