---
title: "Drawbacks of SvelteKitAuth You Should Know"
seoTitle: "SvelteKitAuth's Hidden Drawbacks Unveiled"
seoDescription: "Explore the drawbacks of SvelteKitAuth, from poor documentation to limited community support and challenges in complex scenarios"
datePublished: Sat Jul 13 2024 22:00:05 GMT+0000 (Coordinated Universal Time)
cuid: clyko6zdf000009mcgaz6619r
slug: drawbacks-of-sveltekitauth-you-should-know
tags: disadvantage, sveltekitauth, authjs

---

This is the final chapter of the series Comprehensive Guide to SvelteKitAuth: Secure Authentication for SvelteKit Apps. In the past 14 chapters, we have explored various use cases, how-to guides, and the advantages of using SvelteKitAuth as the authentication library for SvelteKit. In this article, I will highlight a few disadvantages or shortcomings of SvelteKitAuth that must be considered as well.

1. **Complex Scenarios Workarounds:** If your project demands intricate authentication flows or data handling beyond basic login/logout, you might find yourself working around SvelteKitAuth's core functionality, leading to less maintainable code. A few examples of complex scenarios that are not directly possible to work with SvelteKitAuth are:
    
    * *Communication between client and server and vice-versa*: If you want to update data from client to server in real-time, it is not possible to do it with SvelteKitAuth.
        
    * *Sharing sessions like having multiple preview URLs*: If your project requires multiple preview URLs (like in the case of Vercel), that won't be possible as we cannot customize callbackUrl, making it impossible to share sessions on applications deployed on different domains.
        
2. **Limited updates for SvelteKitAuth:** Authjs is the successor to NextAuth, which was designed only for Next.js. Although Authjs aims to be framework agnostic, its main focus is still on Next.js. In SvelteKitAuth, there are many functionalities and configurations that cannot be done, which are possible and can be easily done in Next.js.
    
3. **Poor Documentation:** While working on SvelteKitAuth, I had to frequently switch from the documentation of [**NextAuth**](https://next-auth.js.org/) and [**Authjs**](https://authjs.dev/) as the latter did not have all the necessary information. Recently, they did a major update on the documentation front, but the quality and depth from the SvelteKitAuth perspective are still very poor.
    
4. **Limited community support**: The community support for SvelteKitAuth is almost negligible. To add to the pain, the authors (or maintainers) of this project have stressed multiple times over different threads that it is their side project and they have limited time to invest. So if you come across a potential blocker, due to limited community support and limited support from maintainers, you could find yourself stuck. Note: They have recently started a Discord community and also try to stay active on Twitter, but support for SvelteKitAuth still remains sparse!
    
5. **Developer Experience:** If I speak from my personal experience, I've been one of the early adopters of SvelteKitAuth. I started when we had version *v0.2* and worked until *v1.0*, which is almost one year of working with SvelteKitAuth. This is the time when I realized that SvelteKitAuth is not suitable for my enterprise web application because of the reasons described above. I then switched over to [**Lucia**](https://lucia-auth.com/) and the experience so far has been outstanding.
    

In conclusion, while SvelteKitAuth offers a range of features and benefits for authentication in SvelteKit applications, it is not without its drawbacks. The need for workarounds in complex scenarios, limited updates and focus on Next.js, poor documentation, and minimal community support can pose significant challenges. These factors can make it less suitable for enterprise-level applications or projects with intricate authentication requirements. My personal experience with SvelteKitAuth highlighted these limitations, leading me to switch to Lucia, which has proven to be a more robust solution for my needs. As with any technology, it's essential to weigh the pros and cons to determine the best fit for your specific project requirements.