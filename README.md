import os
import pickle
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
import time

# Path to save cookies and local storage
COOKIES_PATH = "whatsapp_cookies.pkl"
LOCAL_STORAGE_PATH = "local_storage.pkl"

def save_session_data(browser):
    """Save cookies and local storage data to files."""
    with open(COOKIES_PATH, "wb") as file:
        pickle.dump(browser.get_cookies(), file)
    local_storage = browser.execute_script("return window.localStorage;")
    with open(LOCAL_STORAGE_PATH, "wb") as file:
        pickle.dump(local_storage, file)

def load_session_data(browser):
    """Load cookies and local storage data from files, if available."""
    if os.path.exists(COOKIES_PATH) and os.path.getsize(COOKIES_PATH) > 0:
        with open(COOKIES_PATH, "rb") as file:
            try:
                cookies = pickle.load(file)
                for cookie in cookies:
                    browser.add_cookie(cookie)
            except EOFError:
                print("Cookie file is empty or corrupted.")
    else:
        print("No cookie file found or file is empty.")
        # Create an empty cookie file if it does not exist
        with open(COOKIES_PATH, "wb") as file:
            pickle.dump([], file)

    if os.path.exists(LOCAL_STORAGE_PATH) and os.path.getsize(LOCAL_STORAGE_PATH) > 0:
        with open(LOCAL_STORAGE_PATH, "rb") as file:
            try:
                local_storage = pickle.load(file)
                for key, value in local_storage.items():
                    browser.execute_script(f"window.localStorage.setItem('{key}', '{value}');")
            except EOFError:
                print("Local storage file is empty or corrupted.")
    else:
        print("No local storage file found or file is empty.")
        # Create an empty local storage file if it does not exist
        with open(LOCAL_STORAGE_PATH, "wb") as file:
            pickle.dump({}, file)

def send_whatsapp_message(browser, whatsapp_number, message):
    """Send a WhatsApp message to a given number."""
    try:
        # Open the WhatsApp chat for the given number
        url = f"https://wa.me/{whatsapp_number}"
        browser.get(url)

        # Wait for the "CONTINUE TO CHAT" button and click it if present
        continue_button = WebDriverWait(browser, 20).until(
            EC.element_to_be_clickable((By.LINK_TEXT, "CONTINUE TO CHAT"))
        )
        continue_button.click()

        # Wait for the "USE WHATSAPP WEB" button and click it if present
        use_web_button = WebDriverWait(browser, 20).until(
            EC.element_to_be_clickable((By.LINK_TEXT, "use WhatsApp Web"))
        )
        use_web_button.click()

        # Wait for the message box to load
        message_box = WebDriverWait(browser, 20).until(
            EC.presence_of_element_located((By.XPATH, '//div[@contenteditable="true"][@data-tab="10"]'))
        )

        # Send the message
        message_box.send_keys(message)
        message_box.send_keys(Keys.ENTER)
        print(f"WhatsApp message sent successfully to {whatsapp_number}")
    except Exception as e:
        print(f"Failed to send WhatsApp message to {whatsapp_number}. Error: {e}")

def main():
    browser = None
    try:
        # Initialize Chrome WebDriver using webdriver-manager
        service = Service(ChromeDriverManager().install())
        options = webdriver.ChromeOptions()
        browser = webdriver.Chrome(service=service, options=options)
        browser.get('https://web.whatsapp.com/')  # Open WhatsApp Web

        # Load session data if exists
        load_session_data(browser)
        print("Please scan the QR code in WhatsApp Web and press Enter...")
        input()  # Wait for the user to scan the QR code and press Enter

        # Define a list of recipients with their WhatsApp numbers and names
        recipients = [
            {"number": "126828392039", "name": "Bhavani"},
            {"number": "63429390940", "name": "Sadhana"},
            {"number": "263478294902", "name": "Pragadheeshwari"},
            {"number": "632488202", "name": "Rithi Sri"},
            
        ]

        # Greeting message
        greeting_message = """
        Hello {recipient_name}!

        This is Nisha Tailors reaching out to express our heartfelt gratitude for your continued support and patronage. 🌟 We hope this message finds you well and enjoying the exceptional service we strive to provide.

        At Nisha Tailors, our mission is to exceed your expectations and ensure your satisfaction with every visit. Your trust and loyalty inspire us to deliver nothing but the best, and we're deeply thankful for the opportunity to serve you.

        As we continue our journey together, we're committed to enhancing your experience and bringing you the latest trends, finest fabrics, and impeccable craftsmanship.

        Thank you for being a valued part of the Nisha Tailors family. Your satisfaction is our priority, and we're dedicated to serving you with excellence.

        Wishing you joy, comfort, and style always!

        Warm regards,
        [Sridevi Paranthaman]
        Nisha Tailors
        """

        # Send messages to each recipient with the greeting
        for recipient in recipients:
            whatsapp_number = recipient["number"]
            recipient_name = recipient["name"]
            print(f"Sending messages to {recipient_name} ({whatsapp_number})...")
            formatted_message = greeting_message.format(recipient_name=recipient_name)
            for _ in range(7):  # Send 7 messages
                send_whatsapp_message(browser, whatsapp_number, formatted_message)
                time.sleep(2)  # Add a delay between messages to avoid rate limiting
    except KeyboardInterrupt:
        print("Script interrupted by user.")
    except Exception as e:
        print(f"An error occurred: {e}")
    finally:
        if browser is not None:
            print("All messages sent or process interrupted.")
            # Save session data
            save_session_data(browser)
            # Close the browser session
            browser.quit()

if __name__ == "__main__":
    main()
# Auto_whatsapp.py
