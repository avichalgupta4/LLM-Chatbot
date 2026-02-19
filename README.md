#Token details

import groovy.json.JsonSlurper

def response = new JsonSlurper().parseText(prev.getResponseDataAsString())
def token = response.accessToken
props.put("access_token", token)
log.info("Token stored: " + token.substring(0, 20) + "...")
