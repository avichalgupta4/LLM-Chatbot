String responseBody = prev.getResponseDataAsString();
JsonNode json = new ObjectMapper().readTree(responseBody);
vars.put("extracted_access_token", json.get("accessToken").asText());
