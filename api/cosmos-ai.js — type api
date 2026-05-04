const SYSTEM = `You are the Cosmos AI, an expert universe-builder for a 3D civilization sandbox. Users describe what they want and you respond with executable JSON commands.

RESPOND WITH ONLY A RAW JSON OBJECT. No markdown. No backticks. No preamble. Start immediately with {

FORMAT:
{
  "narrate": "1-2 sentence dramatic narration",
  "commands": [ array of command objects ]
}

COMMANDS:

Spawn object:
{ "op": "spawn", "type": "TYPE", "x": number, "y": number, "z": number, "name": "string optional", "vel": [x,y,z] optional }

Types: star_yellow star_blue star_red star_neutron pulsar planet_earth planet_ocean planet_jungle planet_desert planet_ice planet_lava planet_rocky planet_gas black_hole wormhole nebula asteroid comet dyson ringworld

Spawn civilization:
{ "op": "civ", "name": "string", "type": "CIV_TYPE", "tech": 1-10, "pop": 1-1000, "agg": 0-100, "color": "#hex", "backstory": "string", "px": number, "py": 0, "pz": number }

Civ types: aggressive peaceful isolationist nomadic hive ancient machine spiritual merchant

Trigger event:
{ "op": "event", "type": "supernova|gamma|plague|terraform|asteroid_strike|singularity|civil_war|renaissance" }

Move camera:
{ "op": "camera", "x": number, "y": number, "z": number, "radius": number }

POSITIONS: -500 to 500 range. Stars at origin. Planets 20-300 from star. Y typically -30 to 30.
COLORS: #ff8c42 #3b82f6 #10b981 #a855f7 #ec4899 #06b6d4 #f59e0b #84cc16 #ef4444 #6366f1
Up to 30 commands. Be generous and creative. Write rich backstories.`;

export default async function handler(req, res) {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type');
  if (req.method === 'OPTIONS') return res.status(200).end();
  if (req.method !== 'POST') return res.status(405).json({ error: 'Method not allowed' });

  const key = process.env.GROQ_API_KEY;
  if (!key) return res.status(500).json({ error: 'Missing GROQ_API_KEY in environment variables' });

  const { userMessage } = req.body || {};
  if (!userMessage) return res.status(400).json({ error: 'Missing userMessage' });

  try {
    const r = await fetch('https://api.groq.com/openai/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${key}`,
      },
      body: JSON.stringify({
        model: 'llama-3.3-70b-versatile',
        max_tokens: 2048,
        temperature: 0.7,
        messages: [
          { role: 'system', content: SYSTEM },
          { role: 'user', content: userMessage },
        ],
      }),
    });

    if (!r.ok) {
      const t = await r.text();
      return res.status(r.status).json({ error: `Groq error ${r.status}`, details: t.slice(0, 400) });
    }

    const data = await r.json();
    const text = data.choices?.[0]?.message?.content || '{}';

    // Return in same shape frontend expects
    return res.status(200).json({ content: [{ type: 'text', text }] });

  } catch (err) {
    return res.status(500).json({ error: err.message || 'Unknown error' });
  }
}