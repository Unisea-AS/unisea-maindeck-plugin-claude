---
name: maindeck-mcp
description: "Minimal working guide for the UniSea Maindeck MCP server: core terms, phases, statuses, tags, cost item rules, allowed HTML and how to call the tools efficiently and safely. Use it before any Maindeck MCP call, whenever the user asks about their Maindeck projects, vessels, work orders, cost items, owner supply, actions, tender status or activity updates, or says 'the project', 'the WO', 'the spec' or 'the yard quote', even if they do not mention the MCP."
---

# Maindeck MCP: the essentials

UniSea Maindeck plans and runs dry-docking and ship repair projects: write the specification, tender it to
shipyards, run the yard visit, close against the yard's invoice. The MCP acts **as the signed-in user**, with their
role, in one organization. It has **no delete tools**.

## 1. Golden rules

1. **Confirm every write** with the user first. Nothing can be deleted, so a wrong record stays. Fix mistakes by
   renaming or changing the record, or by setting it to `cancelled`.
2. **Resolve before acting.** `list_vessels` then `list_projects` (with `vessel_id`) to get the project. Never guess ids.
3. **Never hardcode tags.** Tags are configured per organization. Call `list_work_order_tags` first and use the
   `complete_key` (for example `executed_by_yard`).
4. **Count, do not page.** Use `count_only=true` or `total_count` instead of fetching long lists.
5. **Relay error messages** as they come. Do not retry blindly. "Not found" also means "no access".
6. **Change phases in the web app.** No MCP tool sets the phase, because the web app runs checks and
   cost steps when the phase changes. Tell the user to change the phase there.
7. **Do not invent data.** If a tool does not return it, say so and point to the web app (section 7).

## 2. Terms

| Term | Meaning |
|---|---|
| Organization | The customer account ("team" in the API) |
| Vessel | A ship ("ship" in the API). Has many projects over time |
| Project | One docking or repair job on one vessel. Has a phase and a survey type |
| Group | Flat, ordered section of a project that holds work orders |
| Work order | One job: specified, quoted, executed and paid. Two words, WO only in tables |
| SFI field (`sfi_code`) | A job code. Follow the project's existing numbering; never "correct" it or derive the group from it |
| Specification | The work order description (`content`), written to the party doing the work |
| Executed by | Key tag category: Yard, Specialist or Crew. Yard puts the work order in the tender |
| Cost item | A scope line the yard prices (quantity, unit, type). Carries **no price** |
| Estimate | One typed budget figure per work order. Not the sum of cost items |
| Yard candidate | A yard invited to quote in Yard selection |
| Owner supply | What the owner buys outside the yard's quote (spares, paint, service engineers) |
| Action | A to-do on a project or a work order |
| Activity update | Progress report with notes and photos; sets the work order's progress |
| Variation order | Work added from Execution onward |

Say **vessel** and **organization** to users. Write **Maindeck** with a lowercase d.

## 3. Phases

- With a yard: `kick_off` > `planning` > `yard_selection` > `pre_execution` > `execution` > `closing` > `completed`
- Without a yard: `kick_off` > `planning` > `execution` > `closing` > `completed`
- The web app moves a project one phase forward or back. The MCP cannot change the phase.

- Kick-off to Pre-execution: plan. Work orders use **`planning_status`**.
- Execution onward: actual. Work orders use **`execution_status`**.
- `get_project_stats` tells which status field is active.

## 4. Statuses

- `planning_status`: `spec_in_progress`, `pending_approval`, `requires_additional_info`,
  `additional_info_provided`, `spec_approved`, `cancelled`. Do not use the legacy values `not_started`,
  `in_progress`, `approved`, `sent_for_quotation`, `completed`.
- `execution_status`: `not_started`, `in_progress`, `delayed`, `pending_approval`, `completed`, `cancelled`.
- `cancelled` is the only way to take a work order out of scope.
- Progress above 0 sets In progress; progress 100 sets Completed.
- `project_status` (health, not phase): `on_schedule`, `attention_required`, `behind_schedule`.

## 5. Writing rules

**Description formatting (CKEditor)**

Users read and edit descriptions in the web app's CKEditor. Write `content` in the same shape CKEditor produces,
minus the attributes the MCP rejects, so the text looks and edits exactly like text typed in the editor.

- **Structure:** four sections, each under an `<h2>` heading:
  1. **Scope:** what is to be done, on which equipment or location. Use exact names and IMO or SFI references
     where known.
  2. **Specification:** materials, standards and method.
  3. **Access / prerequisites:** staging, permits, isolation, owner supply.
  4. **Deliverables:** reports, certificates and photos expected on completion.
- **Paragraphs:** one `<p>` per paragraph. `<br>` only for a line break inside a paragraph. No empty paragraphs
  and no `&nbsp;` for spacing. `<blockquote>` is allowed.
- **Headings:** `<h1>` to `<h6>` are allowed. Use `<h2>` for the four sections. Plain text inside headings, no
  `<strong>`.
- **Inline:** bold `<strong>` or `<b>`, italic `<em>` or `<i>`, underline `<u>`. No nested emphasis.
- **Lists:** `<ul>` or `<ol>` with plain text in each `<li>`, no `<p>` inside `<li>`. A nested list goes inside its
  parent `<li>`.
- **Tables:** `<table><thead><tr><th>..</th></tr></thead><tbody><tr><td>..</td></tr></tbody></table>`. Always a
  header row, plain text in cells, no `figure`, caption, widths or merged cells.
- **Links:** `<a href="...">text</a>` with an http, https, mailto or tel link, or a relative link. `title` is the
  only other attribute allowed.
- **The MCP rejects:** any other tag, such as `div`, `span`, `figure` or `img`, and any other attribute, such as
  `style`, `class` or `data-*`. The editor adds its own wrappers and ids when a user next saves; that is expected.
- **Do not send Markdown or `&nbsp;`.** The MCP accepts them, but the editor shows Markdown as plain symbols, and
  `&nbsp;` is not how the editor spaces text.
- **Editing an existing description:** keep the wording and structure, but rebuild it as clean HTML. Existing
  markup carries editor attributes (`data-list-item-id`, `<figure class="table">`) and is rejected if sent back
  unchanged.

**Description content**
- Address the executing party, be specific, no placeholders, do not repeat what other fields already hold.
- Write for the yard estimator who quotes the job and the crew who does it. Use the imperative ("Blast and coat",
  "Renew gasket"). No marketing language and no hedging.
- Cost-driving figures (areas, lengths, hours) go in cost item quantities, not in the text.
- Keep the language of the existing description.

**Cost items (`add_cost_item`)**
- Only on work orders tagged Executed by: Yard.
- Always pass `cost_type`, `quantity` and, for `per_unit`, `unit`. The defaults break the rules.
- `lump_sum`: quantity 1, no unit. Quantity 0 only for optional or contingency pricing.
- Staging is its own line, per m3.
- Before Execution, a new cost item is copied to **every yard candidate**. Check with the user if yards are already
  invited.
- **Keep the returned id.** No tool lists cost items or actions later; report ids back to the user.

## 6. Recipes

**Create a work order**
1. Resolve the project; `list_project_groups`; `list_work_order_tags`.
2. `create_work_order` with `name`, `group_id`, `content` and `sfi_code` in the project's numbering.
3. `tag_work_order`: Executed by first, then other categories that apply.
4. Yard work only: `add_cost_item` per line (section 5).
5. Report the work order slug and cost item ids.

**Counting**
- Projects in a phase: `list_projects` + `phase` + `count_only=true`.
- Work orders per status: `list_work_orders` + `group_by=status`.
- Work orders with one tag: `list_work_orders` + `tags=[<complete_key>]` + `count_only=true`. Treat `group_by=tag`
  as indicative only.
- Owner supply per category: `list_owner_supply` + `group_by=category`.

**Project overview:** `get_project`, `get_project_stats`, `get_tender_status` (when tendering),
`list_activity_updates` with `from`.

**Tool quirks**
- `get_work_order` returns no tags, cost items or actions. Check a tag with the `tags` filter.
- Project, work order and vessel accept id or slug. All other ids need the UUID.
- Group and account code cannot change after creation. Locked work orders reject every edit.
- Dates are ISO 8601 (`2026-09-01`).

## 7. Not possible through the MCP

Point the user to the web app for: deleting anything; reading existing cost items or listing actions; editing
owner supply; attachments and documents; quote lines, yard selection, exchange rates and budgets; actual, billed
and variation order costs; reports and templates; creating groups, activity updates or comments; external users;
moving work orders between projects.
