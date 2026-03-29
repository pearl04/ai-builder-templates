# AI Self-Eval Pipeline Template
                                                                                                                                                                                                                
  Use a fast, cheap model to QA a powerful model's output before it reaches a human.                                                                                                                            
   
  ## The Pattern                                                                                                                                                                                                
                  
                   ┌──────────────┐                                                                                                                                                                             
                   │   Generate   │  ← Best model (Sonnet, GPT-4, etc.)
                   └──────┬───────┘                                                                                                                                                                             
                          │                                                                                                                                                                                     
                   ┌──────▼───────┐                                                                                                                                                                             
                   │   Evaluate   │  ← Cheap model (Haiku, GPT-4o-mini)                                                                                                                                         
                   └──────┬───────┘                                                                                                                                                                             
                          │
                   Pass?  │                                                                                                                                                                                     
                  ┌───────┴────────┐                                                                                                                                                                            
                  │ Yes            │ No → Retry (max 2x)
                  ▼                ▼                                                                                                                                                                            
           ┌─────────────┐  ┌──────────┐
           │ Human Review │  │ Flag for │                                                                                                                                                                       
           │  (approved)  │  │  manual  │                                                                                                                                                                       
           └─────────────┘  └──────────┘                                                                                                                                                                        
                                                                                                                                                                                                                
  ## Step 1: Define Your instructions                                                                                                                                                                                 
                  
  The eval model needs clear yes/no criteria. Here's a starter instructions for written content:                                                                                                                      
   
  | Criterion | Check | Pass/Fail |                                                                                                                                                                             
  |-----------|-------|-----------|
  | Hook | First sentence is under 10 words and creates curiosity or tension | |                                                                                                                                
  | Voice | Zero red flag phrases from voice profile | |                                                                                                                                                        
  | Structure | Has a clear hook → body → CTA flow | |                                                                                                                                                          
  | Specificity | Contains at least one concrete number, name, or example | |                                                                                                                                   
  | Length | Within target word count (e.g., 50-150 words for a reel caption) | |                                                                                                                               
                                                                                                                                                                                                                
  Customize this for your use case. Code review, email drafts, support responses —                                                                                                                              
  each needs its own set of instructions.                                                                                                                                                                                    
                                                                                                                                                                                                                
  ## Step 2: Write the Eval Prompt                                                                                                                                                                              
   
  Use this as your eval model's system prompt:                                                                                                                                                                  
                  
  ---
  You are a content quality reviewer. You will receive a piece of generated content
  and a set of instructions. Score each criterion as PASS or FAIL. Then give an overall verdict:                                                                                                                             
  APPROVED or RETRY.                                                                                                                                                                                            
                                                                                                                                                                                                                
  Rules:                                                                                                                                                                                                        
  - Be strict. If a criterion is borderline, mark it FAIL.
  - If 1+ criteria fail, verdict is RETRY.                                                                                                                                                                      
  - Provide a one-line fix suggestion for each FAIL.                                                                                                                                                            
  - Do not rewrite the content. Only evaluate.                                                                                                                                                                  
                                                                                                                                                                                                                
  Instructions:                                                                                                                                                                                                       
  [Paste your instructions here]                                                                                                                                                                                      
  ---                                                                                                                                                                                                           
   
  ## Step 3: Choose Your Models                                                                                                                                                                                 
                  
  | Role | Recommended | Why |
  |------|------------|-----|
  | Generator | Claude Sonnet, GPT-4, GPT-4o | Quality of output matters most here |                                                                                                                            
  | Evaluator | Claude Haiku, GPT-4o-mini | Speed and cost matter — eval is pattern matching, not creativity |                                                                                                  
                                                                                                                                                                                                                
  Cost math: If Sonnet costs $3/M tokens and Haiku costs $0.25/M tokens,                                                                                                                                        
  your eval layer adds ~8% to your generation cost while catching 60-80% of quality issues                                                                                                                      
  before a human ever sees them.                                                                                                                                                                                
                  
  ## Step 4: Set Retry Logic                                                                                                                                                                                    
                  
  - Max retries: 2 (more than that = the prompt needs fixing, not more attempts)                                                                                                                                
  - On retry: pass the FAIL reasons back to the generator as feedback
  - After max retries: flag for manual review, don't publish automatically                                                                                                                                      
                                                                                                                                                                                                                
  ## Use Cases Beyond Content                                                                                                                                                                                   
                                                                                                                                                                                                                
  | Use Case | Generator Does | Evaluator Checks |                                                                                                                                                              
  |----------|---------------|-----------------|
  | Code generation | Writes the function | Runs linter rules, checks for common vulnerabilities |                                                                                                              
  | Email drafts | Writes the email | Checks tone, length, CTA presence, no sensitive data |                                                                                                                    
  | Support responses | Drafts the reply | Checks policy compliance, empathy score, accuracy |                                                                                                                  
  | Data extraction | Parses the document | Validates extracted fields against expected format |                                                                                                                
                                                                                                                                                                                                                
  ---                                                           
