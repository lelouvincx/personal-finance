# Personal Finance Analytics

Personal finance BI platform built on Money Manager app data.

## Background

I have been using the Money Manager app for years to track my finances since I started go to work. Up to 2026, it's been 3 years.

![example app](https://media.secondbrain.lelouvincx.com/2026/02/344c2c0fba0700749a5a5196e6bac1e8.png)

The thing I love from Money Manager is that it's very simple. Just a simple UI where I input daily transactions. The hardest thing IMO is to form the habit of recording every transaction (no matter how small it is). Luckily I got it on track.

At the beginning, I always thought about the idea of using my personal finance data to do an analytics project. But never complete it, until now. Also, having a fair amount of data, I think it's a good time to start this project.

This project aims to leverage the data from the Money Manager app to create a personal finance analytics platform that provides valuable insights and visualizations.

Some requirements I have in mind:

- AI-first: When I ask a question in chat, it query data and **give me the personal, customized answer with my own taste**.
- In order to do that there should be a strong semantic foundation.
- Leverage my existing skills such as SQL, Python, data visualizations.
- In the future, be able to connect different data sources such as Logseq, Calendar, Maps to give me a more holistic view of my life and finances.
- **A playground to practice my data skills, as well as learning frontend.**

Detailed data is my personal, sensitive data, so I will not share it here. But I'll share the fake data and the code to generate it, so that you can run the project on your own.

See [AGENTS.md](AGENTS.md) for architecture, decisions, and conventions.

![architecture](https://media.secondbrain.lelouvincx.com/2026/02/56cc15cbc9fe24d3ef49463c76400836.png)

## Roadmap

- [x] Brainsform ([AGENTS.md](./AGENTS.md))
- [x] Initialize the project
- [ ] Data pipeline: ingestion
- [ ] Data pipeline: transformation
- ...

While waiting for progress, you may want to have a sip :D

![](https://media.secondbrain.lelouvincx.com/2026/01/8085eaaf5de0322cedd6246447fa33b4.png)

## License

MIT.
