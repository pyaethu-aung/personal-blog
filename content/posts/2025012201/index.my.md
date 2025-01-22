+++
title = "Hot Reload in Go"
date = '2025-01-22T03:19:00+07:00'
draft = false
lang = "my"
slug = "hot-reload-in-go"
tags = ["development", "go"]
+++

Go လိုမျိုး compiled language အတော်များများမှာ hot reload က natively မပါတာများပါတယ်။ Flutter ကတော့ ချွင်းချက်ပေါ့။ အဲ့တော့ developer တွေအနေနဲ့က code ရေးလိုက် terminal မှာ ပြန်ပြီး recompile လုပ်လိုက်နဲ့က ပုံမှန်ဆို မသိသာပေမယ့် အကြိမ်ရေများလာတဲ့အခါကျ တော်တော်လေး စိတ်အနောင့်အယှက်ဖြစ်ရတယ်။ ကျွန်တော့လိုမျိုး code editor နဲ့ terminal နဲ့ကို သပ်သပ်စီသုံးတတ်တဲ့သူတွေကတော့ ပိုဆိုးမယ်ထင်ပါတယ်။ အဓိကပြဿနာတခုက context switching ပါ။ Complex ဖြစ်ပြီးတော့ nested iteration တွေပါတဲ့ code ကို develop/debug လုပ်နေရတာမျိုးမှာ အာရုံစိုက်ရခက်ပါတယ်။

အကြောင်းကြောင်းကြောင့် Docker container ထဲမှာ compile လုပ်ပြီး စမ်းဖို့လိုလာရင်လည်း လက်ရှိ code ကို volume အနေနဲ့ mount လုပ်နေရတာကလည်း အလုပ်ပိုပါတယ်။

## Air for Hot Reloading
ကျွန်တော်ကတော့ live reloading အတွက် [Air](https://github.com/air-verse/air) ကိုသုံးပါတယ်။ ဘာကောင်းလို့ ညာကောင်းလို့ဆိုတာထက် ex-coworker တရောက်ကစသုံးတာနဲ့ လိုက်သုံးဖြစ်လိုက်တာပါပဲ။ တချို့ အဆင်မပြေတာတွေရှိပေမဲ့လည်း ကျွန်တော်အခုစမ်းရေးနေတဲ့ image processing server အတွက်လည်း Air ပဲသုံးထားလိုက်ပါတယ်။

Install လုပ်တာကလွယ်ပါတယ်။ `go install github.com/air-verse/air@latest` နဲ့ install လုပ်၊ configuration file အနေနဲ့ `.air.toml` file ကို set up လုပ်ပြီး `air -c .air.toml` နဲ့ project ကို run လိုက်ရုံပါပဲ။ `air init` နဲ့ `.air.toml` file ကို initiate လုပ်လို့လည်းရပါတယ်။

## Air in Dockerfile
ကျွန်တော်ကတော့ Docker နဲ့ run ရတာများတဲ့အတွက် Dockerfile ထဲမှာ Air ကို install လုပ်ပြီး [Docker Compose](https://docs.docker.com/compose/) နဲ့ codebase directory ကို mount လုပ်ပြီးသုံးပါတယ်။

Dockerfile
```yaml
FROM golang:1.23.4-bookworm

WORKDIR /app

RUN go install github.com/air-verse/air@latest

COPY . .
RUN go mod download

CMD ["air", "-c", ".air.toml"]
```

docker-compose.yaml
```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    env_file:
      - .env
    ports:
      - 8080:8080
    volumes:
      - ./:/app
```

တခုသတိထားသင့်တာက အခု post ကိုရေးနေတဲ့အချိန်မှာ Air ရဲ့ [latest release](https://github.com/air-verse/air/releases/tag/v1.61.7) က Go v1.23 အောက်ကို support မလုပ်ပါဘူး။ ကိုယ်က အကြောင်းကြောင်းကြောင့် Go version အနိမ့်မှာသုံးဖို့လိုလာရင်တော့ Air version အနိမ့်ကို install လုပ်ဖို့လိုပါမယ်။

## Last But Not Least
`.air.toml` ထဲက `rerun_delay` ရဲ့ value ကို လိုအပ်သလို စက္ကန့်ပိုင်းလောက်ပိုထားဖို့တော့ အကြံပေးချင်ပါတယ်။ မဟုတ်ရင် code changes ကို save လိုက်တာနဲ့ မလိုပဲနဲ့ တန်းပြီး build လုပ်နေပါလိမ့်မယ်။
