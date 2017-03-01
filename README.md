# Twilio Conference Call

In this tutorial we will explore the integration of Twilio to entertain Conference Calls.

## Step 1: Twilio Account

* Create your account at [Twilio.](https://www.twilio.com/)
* After Successful Signup on [Home/Dashboard](https://www.twilio.com/console) , you will find your Credentials.
* Click on [Phone Number Tab](https://www.twilio.com/console/phone-numbers/incoming) and create your first Number.
* Enable [Geographic Permission](https://www.twilio.com/console/voice/settings/geo-permissions) for your Region.

## Step 2: Add and Install Twilio Gem

Modify your Gemfile to include the Devise gem

```
gem 'twilio-ruby', '~> 4.11.1'
```
Then update your gems with

```
bundle install
```
Now restart your server to get the changes

## Step 3: Manage Twilio Credentials for Different Environments

### Create file config/twilio.yml

```
development:
  from: "Twilio Number"
  sid: "Your ACCOUNT SID"
  token: "Your AUTH TOKEN"
  url: "Ngrok Link to Controller"
staging:
  from: "Twilio Number"
  sid: "Your ACCOUNT SID"
  token: "Your AUTH TOKEN"
  url: "Staging Link to Controller"
production:
  from: "Twilio Number"
  sid: "Your ACCOUNT SID"
  token: "Your AUTH TOKEN"
  url: "Production Link to Controller"
```

### Create file config/initializers/twilio.rb

```
path = File.join(Rails.root, "config/twilio.yml")
TWILIO = YAML.load(File.read(path))[Rails.env] || {'sid' => '', 'from' => '', 'token' => '','url' => ''}
```

## Step 4: Create Twilio Client

### Add following to config/application.rb

```
config.after_initialize do
      ::TWILIO = Twilio::REST::Client.new(GLOBAL_CONFIG['sid'], GLOBAL_CONFIG['token'])
end
```

## Step 5: Create Controller twilio_controller.rb

```
  def call_person
    call=::TWILIO.account.calls.create(
                                       url: "#{config['url']}/conference.xml",
                                       to: Number To Be Dialed,
                                       from: config['from'],
                                       status_callback: "Call Back URL",
                                       status_callback_method: "POST",
                                       status_callback_event: ['answered'])
  end
```
Information about the available attributes can be found [here.](https://www.twilio.com/docs/api/rest/call)

## Step 6: Add TWIML Response when user has Received Call

File: twilio/conference.xml.erb

```
  <Response>
  <Say>Welcome Text To Be Played by IVR</Say>
  <Dial timeLimit="Set Time Duration of Call in Seconds">
    <Conference muted= "Used to mute Person on Call" record="record-from-start" startConferenceOnEnter="Start Conference When this User Joins" endConferenceOnExit="End Conference When this Person Leaves" eventCallbackUrl= 'URL where Recording will be posted' >
      @conference_room_no
    </Conference>
  </Dial>
</Response>
```
Action conference
```
  def conference
    @conference_room_no = 'Your Conference Room Number'
    // You can create your own instance vars here to be used as attrs in XML Repsonse.
  end
```

## Step 7: Twilio Dial In Functionality (When a Persons Dials Twilio Number from Cell Phone)

* Open [Phone Number Tab](https://www.twilio.com/console/phone-numbers/incoming).
* Click on the Phone Number to be used.
* In the Voice Section add your URL against 'A CALL COMES IN' and make it an HTTP GET Request. This is the URL on which Twilio will redirect user when it receives a call.

### Create Response TWIML File in the public folder

```
  <Response>
    <Gather action="URL Where User Will Be Redirected" method='GET' timeout="60">
      <Say>PLease enter conference room number and then press pound sign</Say>
    </Gather>
    <Say>We didn't receive any input. Goodbye!</Say>
  </Response>
```
The detail information about the Gather TWIML can be found [here](https://www.twilio.com/docs/api/twiml/gather).

## Step 7: Add Action to Receive User's Input

Action Incoming
```
 def incoming
    @conference_room_no = params["Digits"]
    render "conference"
 end
```
