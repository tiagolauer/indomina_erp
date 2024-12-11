<script setup lang="ts">
import { computed } from "vue";
import { Check, MessageCircle, X } from "lucide-vue-next"; // Adaptado para Vue
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { systemParamsAccount } from "@/parametros";
import { useLoginStore } from "@/stores/login/loginStore";
import { createCheckoutSession, createPortalCaptive } from "@/services/stripe/StripeService";
import { ScToastUtil } from "@/utils/scToastUtil";

const parametros = systemParamsAccount || {
    quota: {
        free: { price: 0, users: 0, services: 0, products: 0, signatures: 0, sales: 0 },
        pro: { price: 0, users: 0, services: 0, products: 0, signatures: 0, sales: 0 },
    },
};

const loginStore = useLoginStore();
const proAccount = computed(() => loginStore.isProAccount);
const isPastDue = computed(() => loginStore.statusAccount === "vencida");
const hasStripeAccount = computed(() => loginStore.statusAccount !== "inativa" && loginStore.dataAccountLogged?.stripeCustomerId);
const showCardFree = computed(() => !proAccount.value && !isPastDue.value);

let isRequestPending = false;

const accessPortalCaptive = async () => {
    if (!hasStripeAccount.value) {
        ScToastUtil.warning("Ocorreu uma falha ao acessar o portal, tente novamente mais tarde!");
        return;
    }
    try {
        const data = await createPortalCaptive(hasStripeAccount.value);
        window.open(data?.data?.url, "_self");
    } catch (error) {
        console.error("Erro no acesso ao portal:", error);
        ScToastUtil.error("Erro ao acessar o portal, tente novamente mais tarde!");
    }
};

const signatureSubscription = async () => {
    if (isRequestPending) return;
    isRequestPending = true;

    if (!hasStripeAccount.value) {
        ScToastUtil.warning("Por favor, recarregue a página e tente novamente mais tarde.");
        isRequestPending = false;
        return;
    }

    try {
        const url = await createCheckoutSession(hasStripeAccount.value);
        window.open(url?.data, "_self");
    } catch (error) {
        console.error("Erro no checkout:", error);
        ScToastUtil.error("Erro ao criar sessão de checkout, tente novamente mais tarde.");
    } finally {
        isRequestPending = false;
    }
};
</script>

<template>
    <div class="container mx-auto max-w-4xl px-4 py-8">
        <div v-if="proAccount" class="mb-8 flex flex-col items-center justify-center text-center">
            <h1 class="text-3xl font-bold text-center">Seu plano atual é PRO!</h1>
            <p class="mt-2 text-lg max-w-md text-muted-foreground">
                O plano PRO é o plano ideal para o seu negócio, tenha todos os recursos ilimitados e suporte total.
            </p>
        </div>

        <div v-else-if="isPastDue" class="mb-8 flex flex-col items-center justify-center text-center">
            <h1 class="text-3xl font-bold text-center">Sua assinatura PRO expirou!</h1>
            <p class="mt-2 text-lg max-w-md text-muted-foreground">
                Com isso, seu plano fica limitado aos recursos do plano Standard. Acesse o link abaixo e gerencie sua
                assinatura
                para voltar a utilizar todos os recursos do PRO.
            </p>
        </div>

        <div v-else class="mb-8 flex flex-col items-center justify-center text-center">
            <h1 class="text-3xl font-bold text-center">Escolha o plano!</h1>
            <p class="mt-2 text-lg max-w-md text-muted-foreground">
                Tenha todos os recursos ilimitados e suporte total assinando o plano PRO.
            </p>
        </div>

        <div class="flex flex-col md:flex-row justify-center gap-4">
            <Card class="md:min-w-[400px]" v-if="showCardFree">
                <CardHeader>
                    <CardTitle
                        class="flex items-center justify-between text-2xl font-bold dark:text-gray-300 text-gray-500">
                        <div>
                            <p class="text-xl font-bold">Plano Standard</p>
                            <p class="text-3xl font-bold">
                                {{ parametros.quota.free.price.toLocaleString('pt-BR', {
                                    style: 'currency', currency:
                                        'BRL'
                                }) }}
                            </p>
                        </div>
                        <Badge variant="default" v-if="showCardFree">Atual</Badge>
                    </CardTitle>
                    <CardDescription>Perfeito para começar</CardDescription>
                </CardHeader>
                <CardContent class="space-y-2">
                    <div class="flex items-center">
                        <Check class="mr-2 h-4 w-4 text-green-500" />
                        <span>Até {{ parametros.quota.free.users }} usuários</span>
                    </div>
                    <div class="flex items-center">
                        <Check class="mr-2 h-4 w-4 text-green-500" />
                        <span>Até {{ parametros.quota.free.services }} serviços</span>
                    </div>
                    <div class="flex items-center">
                        <Check class="mr-2 h-4 w-4 text-green-500" />
                        <span>Até {{ parametros.quota.free.products }} produtos</span>
                    </div>
                    <div class="flex items-center">
                        <Check class="mr-2 h-4 w-4 text-green-500" />
                        <span>Até {{ parametros.quota.free.signatures }} assinaturas</span>
                    </div>
                    <div class="flex items-center">
                        <Check class="mr-2 h-4 w-4 text-green-500" />
                        <span>Até {{ parametros.quota.free.sales }} vendas</span>
                    </div>
                    <div class="flex items-center">
                        <Check class="mr-2 h-4 w-4 text-green-500" />
                        <span>Suporte pelo site</span>
                    </div>
                    <div class="flex items-center">
                        <X class="mr-2 h-4 w-4 text-red-500" />
                        <span class="text-muted-foreground">Assistente de IA</span>
                    </div>
                    <div class="flex items-center">
                        <X class="mr-2 h-4 w-4 text-red-500" />
                        <span class="text-muted-foreground">Recursos avançados</span>
                    </div>
                </CardContent>
                <CardFooter>
                    <Button class="w-full" variant="outline">Seu plano atual</Button>
                </CardFooter>
            </Card>

            <Card class="md:min-w-[400px]">
                <CardHeader>
                    <CardTitle class="flex items-center justify-between text-2xl font-bold">
                        <div>
                            <p class="text-xl font-bold">Plano Pro</p>
                            <p class="text-3xl font-bold dark:text-blue-500 text-blue-600">
                                {{ parametros.quota.pro.price.toLocaleString('pt-BR', {
                                    style: 'currency', currency:
                                        'BRL'
                                }) }}
                            </p>
                        </div>
                        <Badge variant="default" v-if="proAccount">Atual</Badge>
                    </CardTitle>
                    <CardDescription>Recursos ilimitados para seu negócio</CardDescription>
                </CardHeader>
                <CardContent class="space-y-2">
                    <div class="flex items-center">
                        <Check class="mr-2 h-4 w-4 text-green-500" />
                        <span>Até {{ parametros.quota.pro.users === 0 ? '♾️' : parametros.quota.pro.users }}
                            usuários</span>
                    </div>
                    <div class="flex items-center">
                        <Check class="mr-2 h-4 w-4 text-green-500" />
                        <span>Até {{ parametros.quota.pro.services === 0 ? '♾️' : parametros.quota.pro.services }}
                            serviços</span>
                    </div>
                    <div class="flex items-center">
                        <Check class="mr-2 h-4 w-4 text-green-500" />
                        <span>Até {{ parametros.quota.pro.products === 0 ? '♾️' : parametros.quota.pro.products }}
                            produtos</span>
                    </div>
                    <div class="flex items-center">
                        <Check class="mr-2 h-4 w-4 text-green-500" />
                        <span>Até {{ parametros.quota.pro.signatures === 0 ? '♾️' : parametros.quota.pro.signatures }}
                            assinaturas</span>
                    </div>
                    <div class="flex items-center">
                        <Check class="mr-2 h-4 w-4 text-green-500" />
                        <span>Até {{ parametros.quota.pro.sales === 0 ? '♾️' : parametros.quota.pro.sales }}
                            vendas</span>
                    </div>
                    <div class="flex items-center">
                        <Check class="mr-2 h-4 w-4 text-green-500" />
                        <MessageCircle class="mr-2 h-4 w-4 text-green-500" />
                        <span>Suporte por whatsapp</span>
                    </div>
                    <div class="flex items-center">
                        <Check class="mr-2 h-4 w-4 text-green-500" />
                        <span>✨ Assistente de IA</span>
                    </div>
                    <div class="flex items-center">
                        <Check class="mr-2 h-4 w-4 text-green-500" />
                        <span>Recursos avançados</span>
                    </div>
                </CardContent>
                <CardFooter>
                    <Button @click="proAccount ? accessPortalCaptive() : signatureSubscription()" class="w-full">
                        {{ proAccount ? "Gerencie sua assinatura ✨" : "Fazer upgrade para Pro ✨" }}
                    </Button>
                </CardFooter>
            </Card>
        </div>
    </div>
</template>
