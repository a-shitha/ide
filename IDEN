from playwright.sync_api import sync_playwright
import json

def run():
    with sync_playwright() as p:
        # Launch with system Chrome (no need to download Playwright browsers)
        browser = p.chromium.launch(channel="chrome", headless=False)
        context = browser.new_context(storage_state="storage_state.json")
        page = context.new_page()

        # 1. Go to app
        page.goto("https://your-app-url.com", wait_until="networkidle")

        # 2. Authenticate if needed
        if page.locator("input[name='username']").is_visible():
            page.fill("input[name='username']", "your-username")
            page.fill("input[name='password']", "your-password")
            page.click("button[type='submit']")
            page.wait_for_load_state("networkidle")
            context.storage_state(path="storage_state.json")

        # 3. Navigate hidden path to product table
        page.click("text=Open Options")
        page.click("text=Inventory")
        page.click("text=Access Detailed View")
        page.click("text=Show Full Product Table")
        page.wait_for_selector("table")

        # 4. Extract table data
        all_data = []

        while True:
            rows = page.query_selector_all("table tr")
            headers = [h.inner_text().strip() for h in rows[0].query_selector_all("th")]
            for row in rows[1:]:
                cols = row.query_selector_all("td")
                if not cols:
                    continue
                entry = {headers[i]: cols[i].inner_text().strip() for i in range(len(cols))}
                all_data.append(entry)

            # Handle pagination: if "Next" button exists and is enabled, click it
            next_btn = page.locator("text=Next")
            if next_btn.is_visible() and not "disabled" in next_btn.get_attribute("class", ""):
                next_btn.click()
                page.wait_for_timeout(1000)
            else:
                break

        # 5. Save structured JSON
        with open("products.json", "w", encoding="utf-8") as f:
            json.dump(all_data, f, indent=4, ensure_ascii=False)

        browser.close()

if __name__ == "_main_":
    run()