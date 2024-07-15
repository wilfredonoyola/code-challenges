# Leadsyndicate.io Integration with Sources

## Overview

This document outlines the implementation of a service within Leadsyndicate.io that allows the addition of new sources and generates an API key for secure lead submission. This ensures that only authorized sources can send lead data.

## Implementation Steps

### Step 1: Create a Service to Add New Sources and Generate API Keys

**Endpoint to add new sources and generate an API key:**
    
    const express = require('express');
    const crypto = require('crypto');
    const app = express();
    
    const sources = {}; // Here we store sources and their API keys
    
    app.use(express.json());
    
    // Endpoint to add a new source
    app.post('/add-source', (req, res) => {
      const { url } = req.body;
      if (!url) {
        return res.status(400).send('URL is required');
      }
    
      const apiKey = crypto.randomBytes(16).toString('hex');
      sources[url] = apiKey;
    
      res.status(201).send({ url, apiKey });
    });
    
    // Middleware to verify the API key
    app.use((req, res, next) => {
      const apiKey = req.headers['x-api-key'];
      if (!apiKey) {
        return res.status(403).send('API key is required');
      }
    
      const validApiKey = Object.values(sources).includes(apiKey);
      if (!validApiKey) {
        return res.status(403).send('Invalid API key');
      }
    
      // Save the validated API key in the request for later use
      req.validApiKey = apiKey;
      next();
    });
    
    // Endpoint to receive leads
    app.post('/sources/leads', (req, res) => {
      const { name, email } = req.body;
      const referer = req.headers.referer;
    
      // Verify that the API key matches the referer (source)
      const sourceUrl = Object.keys(sources).find(key => sources[key] === req.validApiKey);
    
      if (!sourceUrl || (referer && !referer.includes(sourceUrl))) {
        return res.status(403).send('Invalid API key for the specified referer');
      }
    
      console.log('Lead received:', { name, email });
    
      // Add logic to store the data in the database
      res.status(200).send('Lead received successfully');
    });
    
    app.listen(3000, () => {
      console.log('Server listening on port 3000');
    });
    
### Step 2: Modify the Script on the Third-Party Page
**Script on the third-party page to include the API key::**
    <form id="leadForm">
      <input type="text" id="name" name="name" required placeholder="Name">
      <input type="email" id="email" name="email" required placeholder="Email">
      <button type="submit">Submit</button>
    </form>
    
    <script>
    const API_KEY = 'GENERATED_API_KEY_HERE'; // Replace with the generated API key
    
    document.getElementById('leadForm').addEventListener('submit', function(event) {
      event.preventDefault();
      
      const leadData = {
        name: document.getElementById('name').value,
        email: document.getElementById('email').value,
      };
      
      fetch('https://api.leadsyndicate.io/sources/leads', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'x-api-key': API_KEY // Include the API key in the request
        },
        body: JSON.stringify(leadData)
      })
      .then(response => response.json())
      .then(data => {
        console.log('Success:', data);
      })
      .catch((error) => {
        console.error('Error:', error);
      });
    });
    </script>

### Process Diagram with API Key
- Diagram Description
- Administrator adds a new source in Leadsyndicate.io.
- Leadsyndicate.io generates an API key for the new source.
- The script on the third-party page sends the form data along with the API key.
- Leadsyndicate.io verifies the API key and that it matches the appropriate source before processing the lead data.

<a href="https://ibb.co/s9PMrP1"><img src="https://i.ibb.co/whLx1LQ/leadsyndicate-no-arrows-diagram.png" alt="leadsyndicate-no-arrows-diagram" border="0"></a>

## Final Considerations
- Security: Keep the API keys secure and consider rotating them periodically if necessary.
- Scalability: This solution allows for easily adding new sources and managing API keys for each.
- Maintenance: Facilitates centralized maintenance and management of lead sources.
- This solution provides a secure and scalable method to integrate new lead sources into Leadsyndicate.io, ensuring that only authorized sources can send data using API keys and additional referer checks.

