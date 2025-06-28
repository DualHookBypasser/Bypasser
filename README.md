// pages/api/refresh-session.js

export default async function handler(req, res) {
  const ROBLOSECURITY = process.env.ROBLOSECURITY;

  if (!ROBLOSECURITY) {
    return res.status(500).json({ error: "Missing ROBLOSECURITY in env vars" });
  }

  const fetch = (await import('node-fetch')).default;

  const headers = {
    'Cookie': `.ROBLOSECURITY=${ROBLOSECURITY}`
  };

  // Step 1: Get CSRF token by triggering logout
  let csrfRes = await fetch('https://auth.roblox.com/v2/logout', {
    method: 'POST',
    headers
  });

  const csrfToken = csrfRes.headers.get('x-csrf-token');
  if (!csrfToken) {
    return res.status(403).json({ error: "Failed to retrieve CSRF token" });
  }

  headers['x-csrf-token'] = csrfToken;

  // Step 2: Call a protected endpoint to test session
  const authRes = await fetch('https://users.roblox.com/v1/users/authenticated', {
    headers
  });

  const data = await authRes.json();

  if (!authRes.ok) {
    return res.status(401).json({
      message: "Session expired or invalid",
      error: data
    });
  }

  return res.status(200).json({
    message: "Session is valid",
    user: {
      id: data.id,
      name: data.name,
      displayName: data.displayName
    }
  });
}
