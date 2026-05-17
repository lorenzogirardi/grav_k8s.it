---
title: 'Redefining E-commerce for the Age of Conversational AI'
date: '2025-08-31 18:33'
taxonomy:
    tag:
            - ai
            - chatgpt
            - conversional
            - jsonld
            - mcp
            - ux
---

# Dynamic UX with jsonld and mcp

### Table of Contents

  * Dynamic UX with jsonld and mcp
    * Redefining SEO for the Age of Conversational AI
      * Table of Contents
    * Introduction
    * User Behavior Shift: Conversational AI Discovery Is the New Entry Point
    * Case Study: Upgrading demo-app-eshop
      * Setup
    * Screenshots and What They Show
      * Application overview
    * Technical Deep Dive: Jules’ JSON-LD Integration
    * How JSON-LD Transforms E-Commerce for AI Discovery
      * Immediate AI Advantages
        * Traditional
        * Jsonld
      * Query-Optimized Content
    * Business Impact of AI-Friendly Content
      * Strategic Advantages
        * Traditional
        * Jsonld
      * Future-Proofing Your Commerce Strategy
    * Conclusion: Why This Is Strategic
    * Reflections: Is This Really the Future?
      * What is still missing?



## 

## Introduction

This article began as a classic exploration into JSON-LD and its impact on structured data for e-commerce. However, as research and experimentation progressed, a fundamental realization emerged: what’s actually changing is the entire user experience paradigm. The way end-users interact with digital products is shifting beyond traditional web and app models—toward dynamic, conversational discovery driven by AI.

For years, the typical customer journey followed one of several well-established routes:

  * **Traditional Website Experience:** Customers access product information via desktop or mobile browsers, interacting with a backend-for-frontend (BFF) architecure that serves tailored content.  
[![Traditional Website Experience](/user/images/redefining-ux-for-the-age-of-conversational-ai/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3438302f76313735363635333435392f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6d6973632f747261646974696f6e616c312e706e67)](<https://camo.githubusercontent.com/42e578c6a2a39d3722b804ebcebe4f03f4c5465633e6c5c6d5b19222a8eec789/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3438302f76313735363635333435392f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6d6973632f747261646974696f6e616c312e706e67>)

  * **Mobile App Direct API Access:** Users connected through native apps, directly hitting underlying APIs for transactional or product data.  
[![Mobile App Direct API Access](/user/images/redefining-ux-for-the-age-of-conversational-ai/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3438302f76313735363635333435392f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6d6973632f747261646974696f6e616c322e706e67)](<https://camo.githubusercontent.com/b0345b8805aa2b5a6721a2f8d43367c2a7fab21de066045368172857c598076b/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3438302f76313735363635333435392f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6d6973632f747261646974696f6e616c322e706e67>)

  * **Partner Integrations:** Third-party platforms aggregate or resell products by connecting directly to the e-commerce native APIs, creating new distribution channels.  
[![Partner Integrations](/user/images/redefining-ux-for-the-age-of-conversational-ai/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3438302f76313735363635333435392f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6d6973632f747261646974696f6e616c332e706e67)](<https://camo.githubusercontent.com/6861c048621415b2be5e9e6f4480150118ac5b1aec8c35a08be4d642fc4a65ad/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3438302f76313735363635333435392f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6d6973632f747261646974696f6e616c332e706e67>)  
  
  
  





What’s new (and truly disruptive) is the rise of AI-powered chat interfaces.

  * **Conversational AI Discovery:** The customer asks an AI assistant (via desktop chat, mobile app bot, or integrated partner channel) for product information. Instead of static, pre-built pages, content is dynamically constructed on demand, tailored to the specific user question, intent, and context.  
[![Conversational AI Discovery](/user/images/redefining-ux-for-the-age-of-conversational-ai/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363635333435392f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6d6973632f636861742d64726976656e2d757365722d657870657269656e63652e706e67)](<https://camo.githubusercontent.com/ce6564b4da95a3b820f852537ee3fd785903e836c7b4290a8ebee793d169cce1/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363635333435392f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6d6973632f636861742d64726976656e2d757365722d657870657269656e63652e706e67>)



This shift means that sites must expose product data in ways instantly consumable by AI—not just for classic SEO, but for rich, contextual, real-time answers. JSON-LD becomes the bridge, enabling e-commerce sites to participate fully in the conversational, intent-driven commerce ecosystem.

The days when classic SEO alone could guarantee visibility for products online are ending. As users move to conversational AI platforms—ChatGPT, Gemini, Perplexity, and Mistral—websites must expose data in a way that is instantly machine-readable and context-aware. This involves a shift from simple keywords and descriptions to rich, structured data using [JSON-LD](https://json-ld.org/) and [schema.org](https://schema.org/).

## User Behavior Shift: Conversational AI Discovery Is the New Entry Point

Search is evolving rapidly. More and more users now seek answers, recommendations, and product suggestions directly through conversational platforms—sometimes bypassing traditional search engines altogether. AI chat platforms now provide:

  * **Direct answers** to complex, natural-language queries (for example: “Show me blue Gucci sneakers under £1000 and tell me if they’re in stock”)
  * **Contextual relevance** , understanding and matching user intent far beyond simple keyword search
  * **Instant access** to product details, images,localtion, prices, and offers—sourced from structured, machine-consumable site content



This shift signals a fundamental change: **the future center of digital product discovery is not search (SEO) alone, but Conversational AI Discovery**.  
If product information isn’t provided in a structured, AI-friendly way, it won’t appear in chat results—resulting in missed opportunities and lost sales.

While SEO is still a supporting element in this ecosystem, the primary focus for the next generation of digital commerce is ensuring that products are discoverable, interpretable, and actionable within conversational AI environments.  
Those who adapt will gain visibility and relevance; those who don’t risk disappearing from the main channel where user queries begin.

## Case Study: Upgrading demo-app-eshop

Setup

  * Both versions of the shop were dockerized and served via ngrok to simulate real-world exposure.



  * Live tests were run with AI chat platforms to analyze how each version responded to product queries.



## Screenshots and What They Show

Application overview

[![traditional plp](/user/images/redefining-ux-for-the-age-of-conversational-ai/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535323335312f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f747261646974696f6e616c2f706c702d747261646974696f6e616c2e706e67)](<https://camo.githubusercontent.com/2dbc6009027d20ccea35f6a7ae62870f5678a35dfb6bd97e370bcb32777d7d0c/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535323335312f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f747261646974696f6e616c2f706c702d747261646974696f6e616c2e706e67>)

[![traditional pdp](/user/images/redefining-ux-for-the-age-of-conversational-ai/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535313730312f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f747261646974696f6e616c2f7064702d747261646974696f6e616c2e706e67)](<https://camo.githubusercontent.com/939ecdbc49fb5ce0c2f5d07f71ac61834e667b41f6c70f5a1c74d5f8e1458414/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535313730312f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f747261646974696f6e616c2f7064702d747261646974696f6e616c2e706e67>)

## Technical Deep Dive: Jules’ JSON-LD Integration

**Process Overview:**

  1. **Enhanced Open Graph Metadata** Open Graph tags were upgraded to make previews richer on social and chat platforms.
  2. **Insertion of JSON-LD Structured Data** A script using schema.org/Product was added. This embeds all product details in a format AI models are designed to read.



**Sample Code Snippet:**
    
    
     const jsonLd = {
      "@context": "https://schema.org",
      "@type": "Product",
      name,
      description,
      image: imageUrl,
      offers: {
        "@type": "Offer",
        price: (price / 100).toString(),
        priceCurrency: "GBP",
        availability: "https://schema.org/InStock"
      },
      url: url
    };
    return (
      <div>
        {/* Product UI ... */}
        <script type="application/ld+json" dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }} />
      </div>
    );

  * Updated in `src/app/products/[id]/page.tsx`
  * Every product detail is mapped to its schema property and updated dynamically per page load.
  * Minimal code effort for massive increase in machine and AI visibility.



[![jules](/user/images/redefining-ux-for-the-age-of-conversational-ai/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535363733312f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6d6973632f6a756c65732d70726f6d70742e706e67)](<https://camo.githubusercontent.com/9fc52bd5dac8d3ae212de1567268e774d90b10854f42b133475841d878933e6e/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535363733312f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6d6973632f6a756c65732d70726f6d70742e706e67>)

[![git](/user/images/redefining-ux-for-the-age-of-conversational-ai/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535363733372f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6d6973632f6769746875622d636f6d6d69742d6a736f6e6c642e706e67)](<https://camo.githubusercontent.com/0030c1568a27f9871a06bf48f25492511778424eb31f8ecc4235590160be782f/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535363733372f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6d6973632f6769746875622d636f6d6d69742d6a736f6e6c642e706e67>)

## How JSON-LD Transforms E-Commerce for AI Discovery

Immediate AI Advantages

  * **Direct Information Extraction:** AI engines obtain precise product info (name, price, availability, image) instantly—without network scraping, error, or delay.
  * **Contextual User Matching:** Every question an AI gets (“What’s the price?” “Is it in stock?” “Describe the design and material”) is fully answerable with high accuracy.
  * **Future-Proof and Omni-Channel:** JSON-LD and schema.org format is compatible with chatbots, voice assistants, search engines, social platforms, and future digital channels.
  * **Rapid Development:** As verified in this project, changes require minimal engineering and immediately open the shop to AI-driven referrals.



#### Traditional

[![src pdp traditional](/user/images/redefining-ux-for-the-age-of-conversational-ai/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535323933332f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f747261646974696f6e616c2f7064702d736f757263652d747261646974696f6e616c2e706e67)](<https://camo.githubusercontent.com/784d6e5a5c39e7fa98e73c1650c228d3d62bfaf9b4d195577f1c1ce042389a32/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535323933332f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f747261646974696f6e616c2f7064702d736f757263652d747261646974696f6e616c2e706e67>)

[![schema validation traditional](/user/images/redefining-ux-for-the-age-of-conversational-ai/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535363431382f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f747261646974696f6e616c2f736368656d612d76616c69646174696f6e2d747261646974696f6e616c2e706e67)](<https://camo.githubusercontent.com/1c611d52a58f1fdac39f5c528f1efdc5ec1f3e35bf9d97f2351373fca542ba43/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535363431382f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f747261646974696f6e616c2f736368656d612d76616c69646174696f6e2d747261646974696f6e616c2e706e67>)

[![src pdp traditional](/user/images/redefining-ux-for-the-age-of-conversational-ai/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535363431382f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f747261646974696f6e616c2f726963682d746573742d747261646974696f6e616c2e706e67)](<https://camo.githubusercontent.com/28e422545c60540dc09c1fb9a0c1b6a5f53991089872eca4fd000361d70571de/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535363431382f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f747261646974696f6e616c2f726963682d746573742d747261646974696f6e616c2e706e67>)

#### Jsonld

[![src pdp jsonld](/user/images/redefining-ux-for-the-age-of-conversational-ai/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535363034382f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6a736f6e6c642f7064702d736f757263652d6a736f6e6c642e706e67)](<https://camo.githubusercontent.com/e704e9e548e930635a9e1ee047873c2ff6e052127660487b574dea7b3f81d856/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535363034382f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6a736f6e6c642f7064702d736f757263652d6a736f6e6c642e706e67>)

[![schema validation jsonld](/user/images/redefining-ux-for-the-age-of-conversational-ai/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535363034362f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6a736f6e6c642f736368656d612d76616c69646174696f6e2d6a736f6e6c642e706e67)](<https://camo.githubusercontent.com/71454972b889a9838e97728ae2498ad5bb9ecc9939b574b39b57139571095e7f/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535363034362f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6a736f6e6c642f736368656d612d76616c69646174696f6e2d6a736f6e6c642e706e67>)

[![src pdp jsonld](/user/images/redefining-ux-for-the-age-of-conversational-ai/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535363034352f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6a736f6e6c642f726963682d746573742d6a736f6e6c642e706e67)](<https://camo.githubusercontent.com/5a27c3a2a25db4a00bedadc9ab3fef10cde37908734c6589d318f78fe98f0a9e/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535363034352f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6a736f6e6c642f726963682d746573742d6a736f6e6c642e706e67>)

### Query-Optimized Content

Traditional SEO optimizes for keywords. AI SEO, enabled by JSON-LD, optimizes for **user requests**. Structured data ensures the answer matches the actual question, not what the developer  _hopes_ will be understood.

## Business Impact of AI-Friendly Content

Strategic Advantages

  * **Visibility Where It Matters** Products become “findable” by any platform, not just search engines. AI chat results are now the main driver of discovery.
  * **Conversion-Boosting Experiences** Detailed, relevant, and rich results build trust and drive faster purchase decisions. Users get full answers without friction.
  * **B2B Integration Power** Resellers, affiliates, and partners can integrate products via machine-readable feeds—no custom scraping required.
  * **Analytics Evolution** Organizations can now track conversational engagement, AI-driven sessions, and measure new channels of customer interaction.



#### Traditional

[![perplexity traditional answer](/user/images/redefining-ux-for-the-age-of-conversational-ai/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535333133302f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f747261646974696f6e616c2f706572706c65786974792d636861742d747261646974696f6e616c2e706e67)](<https://camo.githubusercontent.com/fee59bd75bb09b9fac9e59f1efaf2ebb06f57337d3be60b9edd4ac425d9a4c45/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535333133302f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f747261646974696f6e616c2f706572706c65786974792d636861742d747261646974696f6e616c2e706e67>)

#### Jsonld

[![perplexity jsonld answer](/user/images/redefining-ux-for-the-age-of-conversational-ai/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535363034362f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6a736f6e6c642f706572706c65786974792d636861742d6a736f6e2e706e67)](<https://camo.githubusercontent.com/201525928d9c4082c79122dae79336eb3e481078e67e5c42caa610e8613c4e69/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3732302f76313735363535363034362f61692f64796e616d69635f75785f776974685f6a736f6e6c645f616e645f6d63702f6a736f6e6c642f706572706c65786974792d636861742d6a736f6e2e706e67>)

### Future-Proofing Your Commerce Strategy

The retail future is conversational; traditional keywords are giving way to voice and chat recommendations. JSON-LD makes a product catalog “first-class content” in every context—chat, voice, smart home, and social commerce.

## Conclusion: Why This Is Strategic

The experiment proves a simple truth: **Structured data is the new foundation for digital commerce, not only for Google search but for all the intelligent platforms users will increasingly rely on.**

  * **Quick impact for minimal technical effort**
  * **Gains in AI SEO are immediate and measurable**
  * **Strategic advantage in a fast-evolving digital landscape**



Smart businesses will empower their content with AI-centric structure today, gaining visibility, relevance, and adaptability for tomorrow’s shopping journeys.

## Reflections: Is This Really the Future?

So is this the future? Yes and no.

There are enormous new opportunities ahead, but also many new challenges. Conversational platforms like ChatGPT, Claude, Gemini, and Perplexity are quickly becoming the primary interfaces for users—but at present, these interfaces are asynchronous and opaque.

  * **Who is calling your e-commerce?**  
Shop owners never know who is querying their business—there’s no clear user identification when the interaction goes through AI agents.
  * **Unintended side effects of open data**  
The more information we expose, the greater the risk of undesirable consequences. For example, if you publish the number of items in stock, competitors can easily analyze your business performance.
  * **Sensitive data risks**  
There's always the danger of unintentionally exposing sensitive data that should remain private.
  * **WAF (Web Application Firewall) solutions**  
Security solutions for filtering and protecting traffic, like WAFs, become much trickier when the main traffic source is indirect or mediated by AI chat platforms.



### What is still missing?

Realistically, just as search engine monopolies formed around Google, Yahoo, Baidu, or Bing, we’re likely to see a small handful of dominant chat platforms.

These will almost certainly evolve to include sort of collaboration/registration/whatever processes, mechanisms for optimizing the way queries work, and features to pass specific fields enabling a full, end-to-end traceable session—from chat request to final e-commerce transaction.

In short, the frontier is wide open but incomplete:

  * **We need session traceability and user context**
  * **We must balance data openness with business privacy and security**
  * **We should prepare for a future where e-commerce optimization is not just SEO, but also “Chat Optimization” and secure, user-aware integrations with AI platforms**



This is just the beginning, and the conversation—and technology—will keep evolving.
