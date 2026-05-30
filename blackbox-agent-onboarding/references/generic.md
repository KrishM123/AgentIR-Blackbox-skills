# Generic Fallback

Use this path when the repository makes LLM calls but does not match the
supported LangGraph or AI SDK annotation surfaces.

## Goal

Give the user a concrete Blackbox onboarding path without pretending official
annotation-library support exists.

## What To Map

Find and describe:

- the LLM boundaries in the application
- the smallest stable node names the runtime could emit
- where one workflow run begins and ends
- how one stable `rid` can be propagated through that run
- where requests can be redirected to Blackbox

## Runtime Contract

The manual integration target is:

- `POST $BLACKBOX_URL/v1/chat/completions`
- `Authorization: Bearer $BLACKBOX_API_KEY`
- one stable `rid` across the workflow run
- `node-name` for the current LLM boundary

## What Not To Claim

- Do not invent decorators, wrappers, or helper libraries.
- Do not say the repo is compile-ready if there is no official supported
  contract emitter.
- Do not add `agentir.config.json` or compile bootstrap as if it were supported
  automation.

## What To Deliver Instead

Produce a concrete manual onboarding plan:

- which files contain LLM boundaries
- what node names to use
- how `rid` should be created and propagated
- where Blackbox routing should replace direct provider calls
- which gaps remain because the repo is outside official supported annotation
  surfaces
