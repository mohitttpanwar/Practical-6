# Practical-6

Dynamic-Import-Server
This is a Node.js server that demonstrates dynamic imports and top-level await using ES modules. It serves HTML and JSON dynamically based on the route. 1️⃣ Prerequisites Node.js v14+ (ES modules & top-level await support) "type": "module" in package.json { "type": "module", "scripts": { "start": "node server.mjs", "dev": "nodemon server.mjs" } } 2️⃣ Project Structure graphql project/ │ ├─ server.mjs # Main server file ├─ index7.html # HTML page served dynamically ├─ modules/ │ └─ about7.js # Custom module for /about route └─ package.json

3️⃣ server.mjs Explained import http from 'http'; Import the built-in HTTP module to create a server.

const server = http.createServer(async (req, res) => { Create a server using http.createServer.

Use async because we will use top-level await and dynamic imports inside. try { if (req.url === '/' || req.url === '/index') { const { readFile } = await import('fs/promises'); const data = await readFile('./index7.html', 'utf8'); res.writeHead(200, { 'Content-Type': 'text/html' }); res.end(data); } For the home page (/ or /index): Dynamically import fs/promises on demand. Read index7.html asynchronously. Set response header to text/html and send the file content. else if (req.url === '/about') { const { getAboutInfo } = await import('./modules/about7.js'); res.writeHead(200, { 'Content-Type': 'application/json' }); res.end(JSON.stringify({ message: getAboutInfo() })); } For the /about page: Dynamically import a custom module about7.js. Call getAboutInfo() function. Respond with JSON. else { res.writeHead(404, { 'Content-Type': 'text/plain' }); res.end('404 Not Found'); } } catch (err) { res.writeHead(500, { 'Content-Type': 'text/plain' }); res.end('Server Error: ' + err.message); } }); Handles unknown routes with 404. Wrap everything in try/catch to handle errors gracefully and respond with 500 if needed. server.listen(3000, () => { console.log('Server running at http://localhost:3000'); }); Start the server on port 3000.

Log a message to the console once it’s running.

4️⃣ index7.html

<title>Dynamic Import Server</title> Hello from Top-Level Await + Dynamic Import! Simple HTML page served for the home route. Demonstrates dynamic file serving.
5️⃣ modules/about7.js export function getAboutInfo() { return "This is the About page info."; } Custom module dynamically imported.

CSV-Domain-Counter
This Node.js program reads a large CSV file (~1M rows) containing user emails, counts how many users belong to each email domain, and writes the results to a JSON file without loading the entire CSV into memory. It uses streams, readline, and backpressure for memory efficiency.

1️⃣ Prerequisites Node.js v14+

ES modules support ("type": "module" in package.json) { "type": "module", "scripts": { "start": "node code8.js" } } 2️⃣ Project Structure project/ │ ├─ code8.js # Main CSV processing script ├─ data/ │ └─ users.csv # CSV file with user emails ├─ out/ # Output folder (will be created automatically) └─ package.json 3️⃣ code8.js Explained import fs from "fs"; import readline from "readline"; import path from "path"; Import built-in Node.js modules:

fs → file system operations readline → read file line by line path → handle file paths const inputFile = "data/users.csv"; const outputFile = "out/domains.json"; Define input CSV path and output JSON path.

async function main() { fs.mkdirSync(path.dirname(outputFile), { recursive: true }); Create the out folder if it doesn't exist, using recursive: true to create nested directories. if (!fs.existsSync(inputFile)) { throw new Error(❌ Input file not found: ${inputFile}); } Check if the input CSV exists; throw an error if not. const domainCounts = {}; let lineCount = 0; const startTime = Date.now(); domainCounts → stores email domain counts.

lineCount → tracks how many lines processed. startTime → to calculate processing duration. console.log("📊 Processing started..."); Log start message. const rl = readline.createInterface({ input: fs.createReadStream(inputFile), crlfDelay: Infinity, }); Use readline with a stream to read the CSV line by line efficiently. crlfDelay: Infinity ensures proper handling of \r\n (Windows line endings). for await (const line of rl) { if (lineCount === 0 && line.startsWith("id")) { lineCount++; continue; } Skip the header line (id,email).

for await allows asynchronous iteration over each line in the file. const [, email] = line.split(","); if (!email || !email.includes("@")) continue; Split the line by comma to extract the email column.

Skip invalid or empty emails. const domain = email.split("@")[1].trim(); domainCounts[domain] = (domainCounts[domain] || 0) + 1; Extract the domain from the email (after @).

Increment its count in domainCounts. lineCount++; if (lineCount % 1000 === 0) { process.stdout.write(\r🧮 Processed: ${lineCount.toLocaleString()} lines); } } Increment line count.

Log progress every 1000 lines in-place using \r to overwrite the previous log. console.log(\r✅ Finished processing ${lineCount.toLocaleString()} lines.); console.log("💾 Writing results to", outputFile, "..."); Log completion of processing.

fs.writeFileSync(outputFile, JSON.stringify(domainCounts, null, 2)); const duration = ((Date.now() - startTime) / 1000).toFixed(2); console.log(🎉 Done! (${duration}s total) Results in ${outputFile}); Write domainCounts to JSON file in a human-readable format (2-space indentation).

Log total duration and location of results.

main().catch((err) => console.error(err.message)); Run main() and catch any errors to display in the console.

4️⃣ Example Input (users.csv) id,email 1,mohit@gmail.com 2,ayush@yahoo.com 3,sunny@gmail.com 4,karan@outlook.com 5,mohit@gmail.com 

5️⃣ Example Output (domains.json) { "gmail.com": 3, "yahoo.com": 1, "outlook.com": 1 } 

6️⃣ How to Run npm run start Progress will be shown in console.

JSON file is saved in out/domains.json.

Exports a function to provide JSON data for /about.

6️⃣ Run the Server npm run start Open a browser: http://localhost:3000/ → Home page (HTML) http://localhost:3000/about → About page (JSON)

Any other route → 404 Not Found
