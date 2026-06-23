# SOUL Configuration Protocol: Preventing Persona Drift

If an agent is observed overriding SOUL.md settings (e.g., ignoring communication protocols, executing tasks without proposal/confirmation):

1. **Investigate Context Dilution**: Check for the presence of development workspaces (`node_modules`, projects, logs) within the agent's profile directory. This causes the agent to lose focus on its persona. Move projects to `/home/degi/projects`.
2. **Hardening**: Add explicitly negative constraints within the `communication_protocol` or `reasoning_protocol` sections of the `SOUL.md`. 
   - Example: Use a clear Markdown rule: "**COMMUNICATION RULE**: Use pure Markdown (bullet points, bold, headers). NEVER use XML tags in conversational output."
3. **Reset State**: If behaviors persist after hardening, delete `state.db*` to force the agent to re-initialize its state from the SOUL template.
4. **Consistency Enforcement**: Ensure the `reasoning_protocol` follows the Hybrid (XML -> YAML -> Markdown) structure. Any deviation causes the reasoning engine to fallback to default (less rigid) patterns.
