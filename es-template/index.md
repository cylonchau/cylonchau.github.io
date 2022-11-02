# 

## 2.4

```json
{
    &#34;order&#34; : 0,
    &#34;template&#34;: &#34;gsfh-log*&#34;,
    &#34;settings&#34;: {
		&#34;index&#34;: {
			&#34;number_of_shards&#34;: &#34;10&#34;,
			&#34;number_of_replicas&#34;: &#34;1&#34;,
			&#34;refresh_interval&#34;: &#34;5s&#34;
		}
    },
    &#34;mappings&#34;: {
        &#34;_default_&#34;: {
            &#34;_all&#34;: {
                &#34;enabled&#34;: true,
                &#34;omit_norms&#34;: true
            },
            &#34;dynamic_templates&#34;: [
                {
                    &#34;message_field&#34;: {
                        &#34;match&#34;: &#34;message&#34;,
                        &#34;match_mapping_type&#34;: &#34;string&#34;,
                        &#34;mapping&#34;: {
                            &#34;type&#34;: &#34;string&#34;,
                            &#34;index&#34;: &#34;analyzed&#34;,
                            &#34;omit_norms&#34;: true,
                            &#34;fielddata&#34;: {
                                &#34;format&#34;: &#34;disabled&#34;
                            }
                        }
                    }
                },
                {
                    &#34;string_fields&#34;: {
                        &#34;match&#34;: &#34;*&#34;,
                        &#34;match_mapping_type&#34;: &#34;string&#34;,
                        &#34;mapping&#34;: {
                            &#34;type&#34;: &#34;string&#34;,
                            &#34;index&#34;: &#34;analyzed&#34;,
                            &#34;omit_norms&#34;: true,
                            &#34;fielddata&#34;: {
                                &#34;format&#34;: &#34;disabled&#34;
                            },
                            &#34;fields&#34;: {
                                &#34;raw&#34;: {
                                    &#34;type&#34;: &#34;string&#34;,
                                    &#34;index&#34;: &#34;not_analyzed&#34;,
                                    &#34;ignore_above&#34;: 256
                                }
                            }
                        }
                    }
                }
            ],
            &#34;properties&#34;: {
                &#34;@timestamp&#34;: {
                    &#34;type&#34;: &#34;date&#34;
                },
                &#34;time&#34;: {
                    &#34;type&#34;: &#34;date&#34;,
					&#34;format&#34;: &#34;yyyy/MM/DD HH:mm:ss&#34;
                },
                &#34;@version&#34;: {
                    &#34;type&#34;: &#34;string&#34;,
                    &#34;index&#34;: &#34;not_analyzed&#34;
                },
                &#34;host&#34;: {
                    &#34;type&#34;: &#34;string&#34;,
                    &#34;index&#34;: &#34;not_analyzed&#34;
                },
				&#34;message&#34;:{
					&#34;type&#34;: &#34;string&#34;,
                    &#34;index&#34;: &#34;analyzed&#34;
				},
                &#34;method&#34;: {
                    &#34;type&#34;: &#34;string&#34;,
                    &#34;index&#34;: &#34;not_analyzed&#34;
                }
            }
        }
    }
}
```

## 5.6.8

```json
{
    &#34;order&#34; : 0,
    &#34;template&#34;: &#34;logstash-webapi-*&#34;,
    &#34;settings&#34;: {
		&#34;index&#34;: {
			&#34;number_of_shards&#34;: &#34;10&#34;,
			&#34;number_of_replicas&#34;: &#34;1&#34;,
			&#34;refresh_interval&#34;: &#34;5s&#34;
		}
    },
    &#34;mappings&#34;: {
        &#34;_default_&#34;: {
            &#34;_all&#34;: {
                &#34;enabled&#34;: true,
                &#34;omit_norms&#34;: true
            },
            &#34;dynamic_templates&#34;: [
                {
                    &#34;message_field&#34;: {
                        &#34;match&#34;: &#34;message&#34;,
                        &#34;match_mapping_type&#34;: &#34;string&#34;,
                        &#34;mapping&#34;: {
                            &#34;type&#34;: &#34;string&#34;,
                            &#34;index&#34;: &#34;analyzed&#34;,
                            &#34;omit_norms&#34;: true,
                            &#34;fielddata&#34;: {
                                &#34;format&#34;: &#34;disabled&#34;
                            }
                        }
                    }
                },
                {
                    &#34;string_fields&#34;: {
                        &#34;match&#34;: &#34;*&#34;,
                        &#34;match_mapping_type&#34;: &#34;string&#34;,
                        &#34;mapping&#34;: {
                            &#34;type&#34;: &#34;string&#34;,
                            &#34;index&#34;: &#34;analyzed&#34;,
                            &#34;omit_norms&#34;: true,
                            &#34;fielddata&#34;: {
                                &#34;format&#34;: &#34;disabled&#34;
                            },
                            &#34;fields&#34;: {
                                &#34;raw&#34;: {
                                    &#34;type&#34;: &#34;string&#34;,
                                    &#34;index&#34;: &#34;not_analyzed&#34;,
                                    &#34;ignore_above&#34;: 256
                                }
                            }
                        }
                    }
                }
            ],
            &#34;properties&#34;: {
                &#34;@timestamp&#34;: {
                    &#34;type&#34;: &#34;date&#34;
                },
                &#34;@version&#34;: {
                    &#34;type&#34;: &#34;string&#34;,
                    &#34;index&#34;: &#34;not_analyzed&#34;
                },
                &#34;host&#34;: {
                    &#34;type&#34;: &#34;string&#34;,
                    &#34;index&#34;: &#34;not_analyzed&#34;
                },
			  &#34;message&#34;:{
		    		&#34;type&#34;: &#34;string&#34;,
                     &#34;index&#34;: &#34;analyzed&#34;
			   },
                &#34;method&#34;: {
                    &#34;type&#34;: &#34;string&#34;,
                    &#34;index&#34;: &#34;not_analyzed&#34;
                }
            }
        }
    }
}

```
