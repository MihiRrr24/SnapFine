public static long getMinimumSize(List<Integer> payloadSize, List<Integer> cacheA, List<Integer> cacheB, int minThreshold) {
    List<Integer> both = new ArrayList<>();
    List<Integer> onlyA = new ArrayList<>();
    List<Integer> onlyB = new ArrayList<>();
    
    int totalA = 0, totalB = 0;
    
    for (int i = 0; i < payloadSize.size(); i++) {
        int size = payloadSize.get(i);
        int a = cacheA.get(i);
        int b = cacheB.get(i);
        
        if (a == 1) totalA++;
        if (b == 1) totalB++;
        
        if (a == 1 && b == 1) {
            both.add(size);
        } else if (a == 1) {
            onlyA.add(size);
        } else if (b == 1) {
            onlyB.add(size);
        }
    }
    
    if (totalA < minThreshold || totalB < minThreshold) {
        return -1;
    }
    
    Collections.sort(both);
    Collections.sort(onlyA);
    Collections.sort(onlyB);
    
    long[] prefixBoth = new long[both.size() + 1];
    long[] prefixA = new long[onlyA.size() + 1];
    long[] prefixB = new long[onlyB.size() + 1];
    
    for (int i = 0; i < both.size(); i++) {
        prefixBoth[i + 1] = prefixBoth[i] + both.get(i);
    }
    
    for (int i = 0; i < onlyA.size(); i++) {
        prefixA[i + 1] = prefixA[i] + onlyA.get(i);
    }
    
    for (int i = 0; i < onlyB.size(); i++) {
        prefixB[i + 1] = prefixB[i] + onlyB.get(i);
    }
    
    long result = Long.MAX_VALUE;
    
    for (int x = 0; x <= both.size(); x++) {
        int needA = Math.max(0, minThreshold - x);
        int needB = Math.max(0, minThreshold - x);
        
        if (needA <= onlyA.size() && needB <= onlyB.size()) {
            long total = prefixBoth[x] + prefixA[needA] + prefixB[needB];
            result = Math.min(result, total);
        }
    }
    
    return result == Long.MAX_VALUE ? -1 : result;
}

















from litellm import completion
from typing import List


class AnswerGenerator:
    """Generates a natural-language answer using retrieved context and an LLM via litellm."""

    def __init__(self, model: str = "claude-haiku-4-5-20251001") -> None:
        """Store the litellm model identifier as an instance attribute."""
        self.model = model

    def build_prompt(self, query: str, context: str) -> str:
        """Format query and context into a prompt string; raise ValueError if query is empty."""
        if not query or not query.strip():
            raise ValueError("Query cannot be empty.")
        
        prompt = f"""You are a helpful HR assistant. Answer the employee's question using ONLY the provided context below. Be concise and factual. If the answer is not in the context, say "I don't have information about that in the current policy documents."

Context:
{context}

Question: {query}

Answer:"""
        return prompt

    def generate(self, query: str, context: str) -> str:
        """Build the prompt, call litellm.completion, and return the answer string; raise ValueError if query is empty."""
        if not query or not query.strip():
            raise ValueError("Query cannot be empty.")
        
        prompt = self.build_prompt(query, context)
        
        response = completion(
            model=self.model,
            messages=[{"role": "user", "content": prompt}]
        )
        
        return response.choices[0].message.content











Analyze the following customer email to determine if all mandatory dispute details are present.

Mandatory Fields:
- name: The customer's full name (usually found in the signature, greeting, or body).
- amount: The specific transaction amount (including currency symbols).
- date: The date the transaction occurred.
- card_last_four: The last 4 digits of the credit/debit card.

Instructions:
1. Identify if each field is present. Be flexible with natural language (e.g., "card ends in 1234" counts for card_last_four).
2. If all fields are found, output exactly: COMPLETE
3. If any fields are missing:
   - Output "MISSING: " followed by the internal field names (name, amount, date, card_last_four).
   - The missing fields MUST be in alphabetical order.
   - Separate fields with a comma and NO spaces (e.g., MISSING: amount,date).
4. Do not provide any conversational text, explanations, or labels.

Customer Email:
{testcase input}
