# LLM-Chatbot

import org.apache.commons.lang3.StringUtils;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.JsonNode;

try {
    String responseBody = prev.getResponseDataAsString();
    
    ObjectMapper mapper = new ObjectMapper();
    JsonNode json = mapper.readTree(responseBody);

    // Strip leading "$." if present — supports both "$.accessToken" and "accessToken"
    String rawPath = vars.get("tokenJsonPath");
    if (rawPath == null || rawPath.isEmpty()) rawPath = "accessToken";
    String cleanPath = rawPath.startsWith("$.") ? rawPath.substring(2) : rawPath;

    // Walk the JSON tree following dot-separated keys
    String[] keys = cleanPath.split("\\.");
    JsonNode node = json;
    for (String key : keys) {
        if (node == null || node.isMissingNode()) break;
        node = node.get(key);
    }

    if (node == null || node.isNull() || node.asText().isEmpty()) {
        vars.put("extracted_access_token", "TOKEN_NOT_FOUND");
        log.error("Token extraction failed. Key path: " + cleanPath + " | Response: " + responseBody.substring(0, Math.min(300, responseBody.length())));
    } else {
        String tokenValue = node.asText();
        vars.put("extracted_access_token", tokenValue);
        log.info("Token extracted successfully. Path: " + cleanPath + " | Length: " + tokenValue.length());
    }

} catch (Exception e) {
    vars.put("extracted_access_token", "TOKEN_NOT_FOUND");
    log.error("Token extraction exception: " + e.getMessage());
}

---

try {
    String token = vars.get("extracted_access_token");

    if (token == null || token.equals("TOKEN_NOT_FOUND") || token.isEmpty()) {
        log.error("=== TOKEN EXTRACTION FAILED. Check tokenJsonPath and token API response. ===");
        throw new Exception("Access token could not be extracted. Aborting test.");
    } else {
        props.put("access_token", token);
        log.info("=== Token stored globally. Length: " + token.length() + " chars ===");
    }

} catch (Exception e) {
    // Re-throw so JMeter marks the setUp sampler as failed and stops the test
    throw e;
}
