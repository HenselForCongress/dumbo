![Dumbo](/assets/dumbo.svg)

<div id="social" align="center">
  <a href="https://henselforcongress.com" target="_blank"><img src="https://img.shields.io/badge/Campaign%20Website-3066BD?style=for-the-badge&logoColor=white" alt="Hensel for Congress Campaign Website"/></a>
  &nbsp; &nbsp; &nbsp;
  <a href="https://secure.anedot.com/hensel-for-congress/github-support" target="_blank"><img src="https://img.shields.io/badge/Donate-3066BD?style=for-the-badge&logo=donate&logoColor=white" alt="Donate to Hensel for Congress"/></a>
  &nbsp; &nbsp; &nbsp;
  <a href="https://github.com/orgs/HenselForCongress/projects/4" target="_blank"><img src="https://img.shields.io/badge/Get%20Involved-3066BD?style=for-the-badge&logoColor=white" alt="Get Involved with Hensel for Congress"/></a>
</div>

This tool generates mock donation data for the SunBeam project to enhance campaign finance transparency.

## Usage

Run the generator with:
```sh
poetry run python sunbeam/donation.py








<div align="center">
  <table border="1" style="border-collapse: collapse; border: 2px solid black; margin-left: auto; margin-right: auto;">
    <tr>
      <td style="padding: 10px;">
        Paid for by Hensel for Congress
      </td>
    </tr>
  </table>
</div>





Make me a non-shitty concise readme for this


A mock data generator for [SunBeam](https://github.com/HenselForCongress/sunbeam)

to run it, poetry run python sunbeam/donation.py

Here is that modules code. Also license AGPL-3.0.


import requests
import time
from faker import Faker
import random
import uuid
import pandas as pd
import yaml
from datetime import datetime, timedelta

def load_config(filepath):
    with open(filepath, 'r') as file:
        config = yaml.safe_load(file)
    return config

def load_zip_data(filepath):
    df = pd.read_csv(filepath, dtype={'zip': str})
    zip_code_dict = df.set_index('zip')[['city', 'state_id']].to_dict('index')
    return zip_code_dict

def load_company_names(filepath):
    with open(filepath, 'r') as file:
        company_names = file.read().splitlines()
    # print("Loaded company names:", company_names)  # Debug statement to verify the content of the list
    return company_names

def select_zip_code(config, zip_code_dict):
    primary_zips = config['sunbeam']['donation']['zip_code']['primary']
    weight = config['sunbeam']['donation']['zip_code']['weight'] / 100
    if random.random() < weight:
        return random.choice(primary_zips)
    else:
        return random.choice(list(zip_code_dict.keys()))

def generate_dummy_data(zip_code, amount, zip_code_dict, days_back, company_names):
    fake = Faker('en_US')
    location = zip_code_dict.get(str(zip_code), {"city": fake.city(), "state_id": fake.state_abbr()})
    city = location["city"]
    state = location["state_id"]
    amount = round(amount, 2)
    fees_amount = round(amount * 0.04 + 0.30, 2)
    net_amount = round(amount - fees_amount, 2)
    company_name = random.choice(company_names)

    # Generate a random date from now to X days in the past
    days_in_past = random.randint(0, days_back)
    random_date = datetime.now() - timedelta(days=days_in_past)
    formatted_date = random_date.strftime('%Y-%m-%d %H:%M:%S UTC')

    return {
        "submission_id": str(uuid.uuid4()),
        "donation": {
            "id": str(uuid.uuid4().hex),
            "donation_project": "",
            "products": [{
                "internal_identifier": str(uuid.uuid4().hex[:4]),
                "name": fake.word().capitalize()
            }],
            "fees": {
                "anedot_fees": {"amount": str(fees_amount)},
                "vendor_fees": []
            }
        },
        "origin": "hosted",
        "commitment_uid": str(uuid.uuid4()),
        "event_amount": str(amount),
        "amount_in_dollars": str(amount),
        "net_amount": str(net_amount),
        "frequency": "once",
        "action_page_id": str(uuid.uuid4()),
        "action_page_name": "New Donation Page",
        "donor_profile_id": "",
        "payment_method_id": str(uuid.uuid4()),
        "created_at": formatted_date,
        "updated_at": formatted_date,
        "address_line_1": fake.street_address(),
        "address_line_2": "",
        "address_city": city,
        "address_region": state,
        "address_postal_code": zip_code,
        "address_country": "US",
        "email": fake.email(),
        "phone": fake.phone_number(),
        "first_name": fake.first_name(),
        "last_name": fake.last_name(),
        "middle_name": fake.first_name(),
        "occupation": fake.job(),
        "employer_name": company_name,
        "title": "",
        "suffix": "",
        "ip_address": fake.ipv4(),
        "recurring": "true",
        "is_recurring_commitment": "true",
        "referrer": "https://secure.anedot.com/example/slug",
        "referrer_to_form": "",
        "custom_field_responses": {
            "opt_one": "Option One"
        },
        "communications_consent_email": "",
        "communications_consent_phone": "",
        "currently_employed": ""
    }


def send_post_request(url, data):
    headers = {'Content-Type': 'application/json'}
    response = requests.post(url, json={"event": "donation_completed", "payload": data})
    return response

def main():
    config = load_config('config.yml')
    zip_code_dict = load_zip_data('assets/zip_codes.csv')
    company_names = load_company_names('assets/companies.csv')
    url = config['sunbeam']['donation']['request']
    run_count = config['sunbeam']['donation']['run_count']
    repeaters = config['sunbeam']['donation']['repeaters']
    min_amount = config['sunbeam']['donation']['amount']['min']
    max_amount = config['sunbeam']['donation']['amount']['max']
    days_back = config['sunbeam']['donation']['time_machine']

    all_data = []
    for _ in range(run_count):
        selected_zip = select_zip_code(config, zip_code_dict)
        random_amount = random.uniform(min_amount, max_amount)
        data = generate_dummy_data(selected_zip, random_amount, zip_code_dict, days_back, company_names)
        all_data.append(data)

    # Select random entries to repeat donations
    repeater_indices = random.sample(range(len(all_data)), min(repeaters, len(all_data)))
    for index in repeater_indices:
        original_data = all_data[index]
        new_data = original_data.copy()  # Make a deep copy of the data
        new_days_in_past = random.randint(0, days_back)
        new_random_date = datetime.now() - timedelta(days=new_days_in_past)
        new_formatted_date = new_random_date.strftime('%Y-%m-%d %H:%M:%S UTC')
        new_data['created_at'] = new_formatted_date
        new_data['updated_at'] = new_formatted_date
        all_data.append(new_data)

    # Send data to server
    for data in all_data:
        response = send_post_request(url, data)
       # print(f"Response Status Code: {response.status_code}, Response Text: {response.text}")
        print(f"Response Status Code: {response.status_code}")
        dynamic_wait_time = random.randint(15, 150)  # Generate a random wait time between 15 seconds and a few minutes
        time.sleep(dynamic_wait_time)

if __name__ == "__main__":
    main()
