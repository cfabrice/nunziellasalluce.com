<template>
  <main class="home" v-if="acf">
    <HeroSection :acf="acf" :animateHeader="animateHeader" :bgImage="bgImage"/>

    <WhoIAm :acf="acf" :animateWho="animateWho"/>

    <div id="work" v-if="$store.state.window && filteredProjects" class="projects">
      <TheWork :filteredProjects="filteredProjects" :acf="acf" :animateWork="animateWork"/>
    </div>

    <TheProcess :acf="acf" :animateProcess="animateProcess"/>

    <TheCapabilities :acf="acf" :animateCapab="animateCapab"/>

    <TheTestimonials
      :acf="acf"
      :testimonials="testimonials"
      :animateTestimonials="animateTestimonials"
    />
  </main>
</template>

<script>
  import debounce from "lodash/debounce";
  import HeroSection from "@/components/Sections/Home/HeroSection";
  import Config from "~/assets/config";
  import get from "lodash/get";
  let scroller, steps;

  export default {
    scrollToTop: true,
    data () {
      return {
        animateHeader: false,
        animateWho: false,
        animateWork: false,
        animateProcess: false,
        animateCapab: false,
        animateTestimonials: false
      };
    },
    components: {
      HeroSection,
      WhoIAm: () => import("@/components/Sections/Home/WhoIAm"),
      TheWork: () => import("@/components/Sections/Home/TheWork"),
      TheProcess: () => import("@/components/Sections/Home/TheProcess"),
      TheCapabilities: () => import("@/components/Sections/Home/TheCapabilities"),
      TheTestimonials: () => import("@/components/Sections/Home/TheTestimonials")
    },
    head () {
      return { title: "Home" };
    },
    async created () {
      const projects = await this.$axios.get(
        Config.wpDomain + Config.api.projects,
        { useCache: false }
      );
      this.$store.commit("setProjects", projects.data);
    },
    async mounted () {
      if (process.client) {
        const home = await this.$axios.get(
          Config.wpDomain + Config.api.homePage,
          { useCache: false }
        );
        this.$store.commit("setHomepage", home.data);
        setTimeout(() => {
          this.animateHeader = true;
          this.handleScroll();
          this.Splitting();
        }, 150);
        if (this.$route.hash) {
          if (process.client) {
            window && window.scrollTo(0, 0);
            setTimeout(() => {
              this.$VueScrollTo.scrollTo(".projects");
            }, 600);
          }
        }
      }
    },
    methods: {
      hideMenu () {
        this.$store.commit("hideMenuBg");
      },
      showMenu () {
        this.$store.dispatch("showMenu");
      },
      handleStepEnter (response) {
        switch (response.index) {
          case 0:
            this.hideMenu();
            this.animateHeader = true;
            break;
          case 1:
            this.animateWho = true;
            break;
          case 2:
            this.animateWork = true;
            break;
          case 3:
            this.animateProcess = true;
            break;
          case 4:
            this.animateCapab = true;
            break;
          case 5:
            this.animateTestimonials = true;
            break;
          default:
            break;
        }
      },
      handleScroll () {
        if (process.client) {
          const step = document.querySelector(".step");

          if (step) {
            if (window.innerWidth > 577) {
              scroller = this.scrollama();
              steps = null;
              steps = scroller
                .setup({
                  step: ".step",
                  offset: 0.6,
                  debug: false
                })
                .onStepEnter(this.handleStepEnter)
                .onStepExit(this.showMenu);

              steps.resize();
              steps.enable();
            } else {
              scroller = this.scrollama();
              steps = null;
              steps = scroller
                .setup({
                  step: ".step",
                  offset: 0.9,
                  debug: false
                })
                .onStepEnter(this.handleStepEnter)
                .onStepExit(this.showMenu);

              steps.resize();
              steps.enable();
            }

            window.addEventListener(
              "resize",
              this.scrollamaResize,
              { passive: true },
              false
            );
          } else {
            setTimeout(() => {
              this.handleScroll();
            }, 600);
          }
        }
      },
      scrollamaResize: debounce(function () {
        const step = document.querySelector(".step");
        if (step && step.length) {
          this.handleScroll();
        }
      }, 150)
    },
    beforeDestroy () {
      if (typeof scroller !== "undefined") {
        scroller.disable && scroller.disable();
      }
      scroller = null;
      steps = null;
      window.removeEventListener("resize", this.scrollamaResize, false);
    },
    computed: {
      homePage () {
        if (this.$store.state.homePage == null) return false;
        return this.$store.state.homePage;
      },
      projects () {
        if (!this.$store.state.projects.length) return false;
        return this.$store.state.projects;
      },
      filteredProjects () {
        if (!this.projects.length) return false;
        const order = get(this.acf, "case_studies.order", []);
        if (order) {
          let filtered = this.projects;
          // remove drafts
          if (process.env.NODE_ENV === "production") {
            filtered = this.projects.filter(project => {
              return project.acf.status === "true";
            });
          }
          return filtered.sort((a, b) => {
            return order.indexOf(a.id) - order.indexOf(b.id);
          });
        } else return null;
      },
      acf () {
        if (this.$store.state.homePage == null) return false;
        return this.$store.state.homePage.acf;
      },
      testimonials () {
        return get(this.acf, "testimonials.testimonials");
      },
      bgImage () {
        if (process.browser) {
          if (this.$store.state.window < 577) {
            return get(
              this.homePage,
              "acf.hero.mobile_bg.sizes.large",
              "https://placehold.it/2048/2048"
            );
          } else if (
            this.$store.state.window > 576 &&
            this.$store.state.window < 1440
          ) {
            return get(
              this.homePage,
              "acf.hero.desktop_bg.sizes.large",
              "https://placehold.it/2048/2048"
            );
          } else
            return get(
              this.homePage,
              "acf.hero.desktop_bg.sizes.ultra",
              "https://placehold.it/2048/2048"
            );
        }
        return get(
          this.homePage,
          "acf.hero.desktop_bg.sizes.large",
          "https://placehold.it/2048/2048"
        );
      }
    }
  };
</script>

<style lang="scss" scoped>
  h1 {
    @include media(sm) {
      max-width: 480px;
      padding-left: 0;
    }
  }

  section {
    margin: $gap / 1.5 0;
    width: 100%;

    @include media(md) {
      margin: $gap * 3 auto;
    }
  }
</style>
