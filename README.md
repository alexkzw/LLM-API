# LLM-API
This project uses LangChain as the LLM API and TogetherAI as the inference provider for LLM prompt expansion, output validation, and manual evaluation.

# Choice of domain and reference document
The first step involves choosing a domain in which to query the LLM, and a reference document containing information about the domain, which is to be used to expand user prompts.

The domain selected for this implementation focuses on analysing the predicted global economic impact of AI on employment, as well as other associated economic topics such as wage inequality, regulatory challenges, regional differences in AI adoption, and industry-level disruption. 

The purpose of focusing on these topics is to understand how AI will influence labour markets, wage structures, regulatory frameworks, and specific industries at both national and global levels. I chose this domain because the economic impact of AI is a rapidly evolving area. This provides a valuable opportunity to observe how the default GPT model, which is trained on data up to a specific cut-off, responds to economic inquiries, and then compare those responses to those generated after incorporating the reference document, allowing me to assess how the reference influences the model’s output.

The reference document being used is “The impact of artificial intelligence – an economic analysis" published by The Treasury New Zealand. This document was specifically chosen due to its concise yet comprehensive analysis of AI’s economic impact within the context of New Zealand while also referencing broader global trends. The document's structured focus on labour market disruptions, regulatory frameworks, and sectoral implications provides a well-rounded foundation for assessing how AIinduced economic shifts may manifest. The document also includes data from academic studies, governmental reports, and expert forecasts, making it a reliable
source for this analysis.

# Prompt expansion and output validation with reference document
The implementation is structured using the LangChain framework, with TogetherAI as the sole inference provider. The objective is to generate, validate, and refine responses based on a reference document, ensuring that the LLM’s outputs remain factually accurate and grounded in the provided content. The implementation comprises three primary components: prompt expansion, validation, and iterative refinement.

## Prompt expansion with few-shot learning
The prompt expansion mechanism is achieved using the initial_prompt_template() function. This template uses the Few Shot Prompt Template from LangChain to guide the LLM using simple, factual question and answers examples from the reference document. The user prompt is expanded to include the full reference document as context and formatted in a way that instructs the LLM to only use information contained in the reference document. The expanded prompt is then sent to TogetherAI’s LLM using a custom TogetherAI_LLM class that integrates with LangChain’s chain system.

## Output validation
The validation mechanism is implemented using the validation_prompt_template() function. This function systematically checks whether each statement in the LLM’s response is fully supported by the reference document. A validation prompt is used to label every factual claim in the LLM’s response as supported, unsupported, or partially supported with justification. This serves as a critical safeguard against potential hallucinations or unsupported claims. 

The run_llm_chain() function serves as a general-purpose executor for all prompt-response chains. It uses LangChain's modern
Runnable interface by composing the prompt template and TogetherAI model using prompt_template | together_llm. The function invokes the composed chain with the provided inputs and returns the LLM's output. This validation mechanism effectively identifies discrepancies between the LLM’s output and the reference document, ensuring that only factually supported information is included in the final response.

## Iterative refinement
The refinement process is implemented using the refinement_prompt_template() function and serves to iteratively improve the LLM’s output by correcting any unsupported or partially supported claims identified during the validation step. This is achieved through a controlled chain-of-thought (CoT) prompting strategy, where the model is explicitly instructed to re-express its response using only the content from the reference document.

After generating an initial answer, the system invokes the validation prompt, which labels each factual statement as supported, unsupported, or partially supported. These validation results are printed to provide transparency and to determine whether
refinement is necessary. If any claim is found to be unsupported or only partially supported, the system enters a refinement loop, prompting the LLM to revise its answer based strictly on the reference document. This iterative refinement cycle continues for up to five iterations, providing a practical balance between factual alignment and computational efficiency. This is especially important due to the token usage limits under TogetherAI’s free API tier. 

If unsupported claims still persist after the fifth iteration, the best attempt is returned along with a printed message explaining that the maximum refinement depth has been reached. Overall, this CoT-based refinement loop helps ensure that final answers are tightly grounded in the reference document, while the validation output also serves as an evaluative tool to determine whether such refinement is truly necessary or whether the initial few-shot prompting already yields sufficient factual accuracy.

# Evaluation
This evaluation assesses whether the LLM's responses are restricted to information from the chosen reference document, The Impact of Artificial Intelligence – An Economic Analysis. The evaluation draws comparisons between the LLM’s default (unreferenced) responses and the responses generated through the reference-guided pipeline for five prompts. It also examines the necessity and effectiveness of the validation mechanism in ensuring factual alignment.

## Prompt 1: What is the predicted global economic impact of AI on employment?
The default LLM response to this prompt was heavily reliant on non-reference material, including predictions from McKinsey, the World Economic Forum, and the International Labour Organisation. These citations are not present in the reference document,
revealing clear hallucination and overreach. The reference-based output, on the other hand, initially contained one unsupported
claim, which triggered refinement. Iterative validation and refinement continued for three iterations. The final answer grounded its claims in supported content from Georgieff & Hyee (2021), Briggs and Kodnani (2023), Chui et al. (2023), and Acemoglu and Restrepo (2019), among others. The final output presented a nuanced synthesis of macroeconomic, microeconomic, and labour perspectives, all traceable to the source. In this case, validation was essential, as the first two drafts contained unsupported or only partially supported claims.

## Prompt 2: How will AI influence wage inequality globally?
The reference-based pipeline generated a concise answer, attributing wage inequality effects to “so-so automation” as described by Acemoglu and Restrepo (2022). The validation process confirmed full support on the first iteration. This illustrates that the
few-shot prompting successfully guided the LLM toward a factually grounded and highly specific response. Here, validation was not strictly necessary but provided an extra layer of assurance.

## Prompt 3: What regulatory challenges are anticipated with AI adoption?
The reference-grounded response accurately captured multiple challenges discussed in the source: the need to revise copyright laws, balance innovation with rights protection, and address data sovereignty, particularly with respect to Māori communities. All claims were supported in the first validation iteration. This demonstrates that the few-shot prompt was sufficient, and the validation confirmed fidelity without requiring refinement.

## Prompt 4: How will AI adoption differ across regions and countries?
The reference-based response identified and cited several well-supported claims: that advanced economies like New Zealand may experience greater short-term productivity gains from AI (Briggs and Kodnani, 2023), that less developed economies may face slower adoption due to structural barriers (Burns, 2009), and that smaller nations might be affected by the concentration of AI development in large multinational firms (Bell et al., 2023). All claims were validated as supported in the first iteration, and
no refinement was necessary.

In this case, the few-shot prompting strategy effectively grounded the model in the correct context and guided it toward relevant, well-attributed insights. The validation mechanism, while not strictly required, played a useful verification role by confirming that no speculative claims were introduced and that each part of the response was traceable to the reference document. This ensured a high degree of confidence in the factual alignment of the final answer.

## Prompt 5: What industries are most likely to be disrupted by AI according to the report?
The grounded response accurately cited Briggs and Kodnani (2023), noting that office support, legal, engineering, science, and business operations are most vulnerable, with 35–46% of tasks potentially automatable. Validation confirmed all claims as supported in the first pass. The initial prompting was sufficient, but the validation step acted as a formal check.

# Effectiveness of the validation process
The validation mechanism proved essential for maintaining factual integrity. In prompt 1, it successfully identified and corrected unsupported or partially supported claims. These refinements improved factual accuracy and prevented speculative language from appearing in the final outputs. Across the five prompts, the validation and refinement mechanism were required in the first instance, confirming its role as a critical safeguard. 

In the remaining four prompts, the few-shot prompting setup alone was effective in steering the LLM toward accurate, reference-aligned outputs. Nevertheless, the validation process served as an important audit layer, reinforcing confidence in the
factual grounding of the answers. Overall, the validation mechanism acts as a necessary backstop against hallucination and overstatement. Even when not strictly needed, it provides transparency into the reliability of the model’s responses. Combined with the iterative refinement chain, this approach ensures a robust balance between response quality and factual accuracy.

# Conclusion
The implementation effectively leverages LangChain’s LLMChain structure, incorporating TogetherAI as the inference provider, to generate, validate, and refine responses based on the reference document. The validation mechanism played a pivotal role in identifying unsupported claims and guiding the refinement process, thereby enhancing factual accuracy and alignment with the reference content. The iterative refinement mechanism demonstrated its utility in correcting unsupported claims, particularly in cases where initial responses diverged from the reference content. 
