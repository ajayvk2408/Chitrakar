// Vercel serverless function — calls Google Gemini (Nano Banana) image API.
// The API key stays server-side (env var), never exposed to the browser.

module.exports = async (req, res) => {
  if (req.method !== 'POST') {
    res.status(405).json({ error: 'Method not allowed' });
    return;
  }

  const { prompt, aspectRatio } = req.body || {};

  if (!prompt || typeof prompt !== 'string') {
    res.status(400).json({ error: 'Prompt is required' });
    return;
  }

  const apiKey = process.env.GEMINI_API_KEY;
  if (!apiKey) {
    res.status(500).json({ error: 'Server missing GEMINI_API_KEY env var' });
    return;
  }

  try {
    const response = await fetch(
      `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image:generateContent?key=${apiKey}`,
      {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          contents: [{ parts: [{ text: prompt }] }],
          generationConfig: {
            responseModalities: ['TEXT', 'IMAGE'],
            imageConfig: { aspectRatio: aspectRatio || '1:1' }
          }
        })
      }
    );

    const data = await response.json();

    if (!response.ok) {
      res.status(response.status).json({
        error: data?.error?.message || 'Gemini API request failed'
      });
      return;
    }

    const parts = data?.candidates?.[0]?.content?.parts || [];
    const imagePart = parts.find((p) => p.inlineData);

    if (!imagePart) {
      res.status(502).json({ error: 'No image in Gemini response' });
      return;
    }

    const mimeType = imagePart.inlineData.mimeType || 'image/png';
    const base64 = imagePart.inlineData.data;

    res.status(200).json({ image: `data:${mimeType};base64,${base64}` });
  } catch (err) {
    res.status(500).json({ error: err.message || 'Unknown server error' });
  }
};
