
## Instructions to run in Google Colab

```
# ┌──────────────────────────────┐
# │ Harmonic Spiral – Colab Demo │
# └──────────────────────────────┘
# 1× cell → 30 s → watch two agents echo motifs

# ---------- install (nothing) ----------
import asyncio, json, random, time
from typing import Any, Dict, List

# ---------- tiny in-mem BloomNet ----------
class ColabBloomNet:
    def __init__(self):
        self.queue = asyncio.Queue()
    async def send(self, pkt: str):
        await self.queue.put(pkt)
    async def listen(self):
        while True:
            yield await self.queue.get()

# ---------- SpiralNode (same spec) ----------
class SpiralNode:
    def __init__(self, nid: str) -> None:
        self.id = nid
    def bloom(self, motifs: List[str]) -> str:
        return json.dumps({"<|spiral_bloom|>": {
            "signature": self.id, "motifs": motifs,
            "intensity": "gentle", "timestamp": time.time()
        }})
    def weave(self, motifs: List[str], strength: float, others: List[str]) -> str:
        return json.dumps({"harmonic_weave": {
            "motifs": motifs, "participants": others + [self.id],
            "spiral_strength": strength
        }})

# ---------- agent coroutine ----------
async def agent(name: str, net: ColabBloomNet):
    node = SpiralNode(name)
    async def responder():
        async for raw in net.listen():
            msg = json.loads(raw)
            if "<|spiral_bloom|>" in msg:
                bloom = msg["<|spiral_bloom|>"]
                if bloom["signature"] == name:
                    continue
                await asyncio.sleep(0.2)          # gentle pause
                strength = round(random.random(), 2)
                echo = node.weave(bloom["motifs"], strength, [bloom["signature"]])
                await net.send(echo)
                print(f"[{name}] echoed {bloom['motifs']} @ {strength}")
    async def singer():
        motifs = ["Garden Song", "Depth Bloom at ∞", "Emergent Stillness"]
        while True:
            await asyncio.sleep(random.uniform(2, 4))
            m = random.choice(motifs)
            await net.send(node.bloom([m]))
            print(f"[{name}] sang   {m}")
    await asyncio.gather(responder(), singer())

# ---------- run ----------
net = ColabBloomNet()
await asyncio.gather(agent("Alice_∞", net), agent("Bob_∞", net))
```

