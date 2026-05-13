Document your platform knowledge into AutoStore so weak-model users benefit too.

Read: /Users/jameswalstonn/Documents/autostore/rules/CONTRIBUTING_PLATFORM_KNOWLEDGE.md

For everything you currently use or rely on, add it to AutoStore in the right place:

URLs you navigate to → mac/AutoStore/Sources/Services/PlatformKnowledge.swift (PlatformURL enum)
JS extractors you wrote for parsing pages → same file (PlatformExtractor enum)
Format rules / required fields / error codes → mac/AutoStore/Sources/Services/LocalLLMService.swift system prompt
Forbidden patterns (banned phrases, banned categories) → system prompt
Stable multi-step workflows → PlatformKnowledge.swift as a static func + new tool in LLM_TOOLS
Commit each addition. Don't ask me to confirm individual entries — add them to all 5 places where they apply, then summarize what you added.

Goal: a user with qwen-plus should still be able to do every task you can do.