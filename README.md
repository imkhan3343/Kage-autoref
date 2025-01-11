import uuid
import os
from email.mime.text import MIMEText
import smtplib

class KageReferralSystem:
    def __init__(self, email_host='smtp.gmail.com', email_port=587, proxy=None):
        self.referral_codes = {}
        self.user_rewards = {}
        self.base_url = "app.chirpwireless.io"
        self.email_host = email_host
        self.email_port = email_port
        self.proxy = proxy  # Proxy configuration, can be None for no proxy

    def generate_referral_code(self, user_id):
        code = str(uuid.uuid4())[:8]
        self.referral_codes[code] = user_id
        return code

    def refer_friend(self, referrer_code, friend_id):
        if referrer_code in self.referral_codes:
            referrer_id = self.referral_codes[referrer_code]
            self.user_rewards[referrer_id] = self.user_rewards.get(referrer_id, 0) + 100  # $CHIRP
            self.user_rewards[friend_id] = self.user_rewards.get(friend_id, 0) + 25000  # Data Chips
            return True
        return False

    def get_referral_link(self, referral_code):
        return f"{self.base_url}/?referral={referral_code}"

    def get_user_rewards(self, user_id):
        rewards = self.user_rewards.get(user_id, 0)
        return f"{rewards} Data Chips" if rewards >= 1000 else f"{rewards} $CHIRP"

    def display_leaderboard(self):
        sorted_users = sorted(self.user_rewards.items(), key=lambda x: x[1], reverse=True)
        for rank, (user_id, points) in enumerate(sorted_users, 1):
            print(f"Rank {rank}: User {user_id} - {self.get_user_rewards(user_id)}")

    def create_email_account(self, username, password):
        """
        Simulates creating an email account, using proxy if specified.
        """
        if self.proxy:
            return f"Email account created for {username} via proxy {self.proxy}"
        else:
            return f"Email account created for {username} without proxy"

    def login_to_email(self, username, password):
        """
        Simulates logging into an email, using proxy if specified.
        """
        if self.proxy:
            return f"Logged into email for {username} via proxy {self.proxy}"
        else:
            return f"Logged into email for {username} without proxy"

    def save_data_to_account_folder(self, user_id, data):
        """
        Saves data to a folder named after the user_id. This assumes data is text.
        """
        folder = f"account_data/{user_id}"
        try:
            if not os.path.exists(folder):
                os.makedirs(folder)
            with open(f"{folder}/data.txt", "w") as f:
                f.write(data)
        except OSError as e:
            print(f"Error saving data for user {user_id}: {e}")

    def automated_user_onboarding(self, user_id, email_username, email_password):
        """
        Automates the process of creating an email, logging in, and saving data.
        """
        referral_code = self.generate_referral_code(user_id)
        email_creation_msg = self.create_email_account(email_username, email_password)
        login_msg = self.login_to_email(email_username, email_password)
        
        user_data = f"""
        User ID: {user_id}
        Email: {email_username}
        Referral Code: {referral_code}
        {email_creation_msg}
        {login_msg}
        """
        
        self.save_data_to_account_folder(user_id, user_data)
        print(f"User data saved for {user_id}")

# Example usage:
# Example with no proxy
kage_referral_system_no_proxy = KageReferralSystem()
kage_referral_system_no_proxy.automated_user_onboarding("user123", "[email protected]", "password123")

# Example with proxy
proxy_config = {"host": "proxy.example.com", "port": 8080}
kage_referral_system_with_proxy = KageReferralSystem(proxy=proxy_config)
kage_referral_system_with_proxy.automated_user_onboarding("user456", "[email protected]", "password123")

# Refer a friend (using the system with proxy)
kage_referral_system_with_proxy.refer_friend(kage_referral_system_with_proxy.generate_referral_code("user456"), "friend789")

# Check rewards and leaderboard using the proxy system
print(kage_referral_system_with_proxy.get_user_rewards('user456'))
kage_referral_system_with_proxy.display_leaderboard()
