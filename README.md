Automated Weekly Newsletter Generator
======================================

An n8n workflow that scrapes five Kenyan news publications every Friday,
uses AI to extract the top headline from each, and writes a witty newsletter
intro in the voice of "Kamau" - the fictional tech editor of Nairobi Daily.
A human approval step gates the final send to the marketing team.


HOW IT WORKS
------------

1. Schedule Trigger
   Fires every Friday at 2:00 PM automatically.

2. Scrape 5 Sources (in parallel)
   - TechCabal          https://techcabal.com
   - Business Daily     https://www.businessdailyafrica.com
   - The Star           https://www.the-star.co.ke
   - Standard Media     https://www.standardmedia.co.ke
   - Kenyan Wall Street https://kenyanwallstreet.com

3. Trim HTML
   Each source's raw HTML is stripped of scripts, styles, and tags.
   Only the first 2,000 characters of clean text are kept to stay
   within the AI model's useful context window.

4. Field Reporter Agents (5x Mistral Small)
   One AI agent per source acts as a "Field Reporter". It reads the
   trimmed text and returns the single most important headline as
   structured JSON: { source, headline }

5. Merge
   All five headlines are combined into one item for the editor.

6. Chief Editor Agent (Mistral Large + Claude Sonnet fallback)
   The editor agent writes the newsletter intro following strict rules:
   - Under 200 words
   - Markdown format with a heading and 1-2 paragraphs
   - Kenyan slang used naturally (Bazeng!, Sawa sawa, Si mchezo, etc.)
   - References the actual source publications by name
   - Warm, witty, locally relevant tone
   Output includes word count, sources used, slang used, and timestamp.

7. Approval by Tech Editor (Human-in-the-Loop)
   A Gmail "Send and Wait" node emails the draft to the editor with a
   one-click approval link. The workflow pauses here until approved.
   The link expires after 24 hours.

8. Log Approved Draft
   Once approved, the draft is written to a Google Sheets spreadsheet
   (NewsletterManager > Drafts tab) with date, sources count, body,
   and model used.

9. Send to Marketing Team
   The approved draft is emailed to the marketing team with full
   metadata: word count, sources, slang, model, generated timestamp,
   and approval status.


AI MODELS USED
--------------

Field Reporters   Mistral Small (mistral-small-latest)
                  Low temperature (0.1), max 600 tokens
                  One instance per source publication

Chief Editor      Mistral Large (mistral-large-latest)
                  Temperature 0.5, max 2000 tokens
                  Fallback: Claude Sonnet 4.5 (Anthropic)


CREDENTIALS REQUIRED
--------------------

To import and run this workflow you will need to set up the following
credentials inside your n8n instance:

  Mistral Cloud API     For all headline extraction and editor agents
  Gmail OAuth2          For the approval email and marketing team send
  Google Sheets OAuth2  For logging approved drafts


SETUP INSTRUCTIONS
------------------

1. Import the workflow JSON into your n8n instance:
   Settings > Import from file > select the .json file

2. Open the workflow and configure credentials:
   - Click each Mistral node and connect your Mistral Cloud API key
   - Click the Gmail nodes and connect your Gmail OAuth2 account
   - Click the Google Sheets node and connect your Sheets OAuth2 account

3. Update the Google Sheets document ID in the "Log Approved Draft" node
   to point to your own spreadsheet. Create a sheet called "Drafts" with
   columns: Date, Succesful Sources, Draft Body, Model Used

4. Update the approval and marketing email addresses in the Gmail nodes
   to your own addresses.

5. Activate the workflow. It will run automatically every Friday at 2 PM.


PUBLICATIONS SCRAPED
--------------------

All five sources are Kenyan media outlets covering technology, business,
and general news. The workflow is designed around the Kenyan market and
the newsletter persona reflects Nairobi's tech community culture.

  TechCabal          Pan-African tech journalism
  Business Daily     Kenya's leading business newspaper
  The Star           General Kenyan news
  Standard Media     One of Kenya's oldest media houses
  Kenyan Wall Street Finance and investment news


NOTES
-----

- The workflow is set to inactive by default. Activate it when ready.
- Mistral Large is used for the final newsletter write because it
  produces more creative, fluent output than Mistral Small.
- Claude Sonnet is wired as a second language model to the Chief Editor
  node and serves as a fallback if Mistral Large fails.
- Each AI agent uses a session-keyed memory buffer scoped to the current
  run so there is no bleed between weekly executions.
