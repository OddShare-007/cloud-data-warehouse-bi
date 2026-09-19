# Design Guidance

## Visual Direction
Use a professional, minimal data-engineering portfolio style: restrained color, generous whitespace, clear hierarchy, and diagrams that explain flow rather than decorate it.

## Color Palette
- Background: white or very light gray.
- Primary: deep navy for headings and infrastructure components.
- Secondary: muted blue for data flow and links.
- Accent: teal or green for successful processing states.
- Warning: amber for cost, security, or operational warnings.

Use color as a supplement, not the only way to communicate meaning.

## Architecture Diagrams
- Use left-to-right flow: source → S3 → Redshift → BI.
- Group resources by AWS boundary, data layer, and user-facing layer.
- Label protocols or mechanisms such as `COPY`, IAM role, SQL, and read-only access.
- Distinguish raw, staging, curated, and presentation layers.
- Avoid tiny text, unnecessary icons, and crossing arrows.
- Include a small security boundary and a cost-control note.

## README
The README should open with a one-line value proposition, a compact architecture image, a feature list, and a quickstart. Follow with prerequisites, deployment, loading, querying, dashboard setup, testing, cleanup, and troubleshooting.

Use short code blocks, consistent heading levels, tables for configuration, and callouts for security and cost. Screenshots should show synthetic data only and should be cropped to the useful interface.

## Portfolio Standard
Show decisions and trade-offs, not just screenshots. Include a data dictionary, sample SQL, CI status, Terraform structure, and a short explanation of what would change at production scale.
